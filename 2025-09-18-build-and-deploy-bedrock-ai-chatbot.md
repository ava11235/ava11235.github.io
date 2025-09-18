# Building a Serverless AI Hiking Chatbot with AWS Bedrock and Claude 3.5 Sonnet

*Learn how to create an intelligent Pacific Northwest hiking assistant using modern serverless architecture*


<img width="1354" height="881" alt="image" src="https://github.com/user-attachments/assets/b560e18c-234b-4032-85e0-b7772ae4f4a3" />


## Introduction

Ever wanted to build an AI-powered chatbot that actually knows what it's talking about? In this tutorial, we'll create a specialized hiking assistant for the Pacific Northwest using Amazon Bedrock's Claude 3.5 Sonnet model, complete with user authentication and a beautiful React frontend.

**What we're building:**
- 🌲 AI chatbot with PNW hiking expertise
- 🔐 Secure user authentication via AWS Cognito
- ⚡ Serverless architecture that scales automatically
- 🌐 Global content delivery via CloudFront
- 💰 Pay-per-use pricing model

**Technologies used:**
- Amazon Bedrock (Claude 3.5 Sonnet)
- AWS Lambda, API Gateway, Cognito
- React TypeScript frontend
- AWS CDK for infrastructure

## Prerequisites

Before we start, make sure you have:
- AWS CLI configured with appropriate permissions
- Node.js 18+ and npm installed
- AWS CDK installed globally: `npm install -g aws-cdk`
- Access to Amazon Bedrock (Claude 3.5 Sonnet model)

## Step 1: Project Structure Setup

First, let's create our project structure:

```bash
mkdir pnw-hiking-chatbot
cd pnw-hiking-chatbot

# Create main directories
mkdir frontend backend infrastructure
mkdir backend/lambda
```

Your project should look like this:
```
pnw-hiking-chatbot/
├── frontend/          # React TypeScript app
├── backend/lambda/    # Lambda function code
└── infrastructure/    # AWS CDK code
```

## Step 2: The Heart - Lambda Function with Bedrock Integration

The core of our chatbot is a Lambda function that calls Amazon Bedrock. Create `backend/lambda/chat-handler.js`:

```javascript
const { BedrockRuntimeClient, InvokeModelCommand } = require("@aws-sdk/client-bedrock-runtime");

const client = new BedrockRuntimeClient({ region: process.env.AWS_REGION });

exports.handler = async (event) => {
  const headers = {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Content-Type,Authorization',
    'Access-Control-Allow-Methods': 'POST,OPTIONS'
  };

  // Handle CORS preflight
  if (event.httpMethod === 'OPTIONS') {
    return { statusCode: 200, headers, body: '' };
  }

  try {
    const body = JSON.parse(event.body);
    const userMessage = body.message;

    if (!userMessage) {
      return {
        statusCode: 400,
        headers,
        body: JSON.stringify({ error: 'Message is required' })
      };
    }

    // 🎯 This is where the magic happens - Prompt Engineering!
    const prompt = `You are a knowledgeable Pacific Northwest hiking assistant. 
You specialize in trails, gear, weather, and safety for hiking in Washington and Oregon.

Key areas of expertise:
- Popular trails: Mount Rainier, Olympic Peninsula, North Cascades, Columbia River Gorge
- Seasonal considerations: Rain, snow, mud seasons
- Essential gear: Rain protection, layers, sturdy boots, navigation
- Safety: Wildlife (bears, cougars), weather changes, creek crossings
- Trail conditions and permits

Provide helpful, accurate, and practical advice. Keep responses conversational and under 200 words.

User question: ${userMessage}

Response:`;

    // 🤖 Call Claude 3.5 Sonnet via Bedrock
    const command = new InvokeModelCommand({
      modelId: "anthropic.claude-3-5-sonnet-20241022-v2:0",
      contentType: "application/json",
      accept: "application/json",
      body: JSON.stringify({
        anthropic_version: "bedrock-2023-05-31",
        max_tokens: 500,
        messages: [{ role: "user", content: prompt }]
      })
    });

    const response = await client.send(command);
    const responseBody = JSON.parse(new TextDecoder().decode(response.body));
    
    return {
      statusCode: 200,
      headers,
      body: JSON.stringify({
        response: responseBody.content[0].text,
        timestamp: new Date().toISOString(),
        messageId: Date.now().toString()
      })
    };

  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      headers,
      body: JSON.stringify({ 
        error: 'Sorry, I encountered an issue. Please try again.',
        timestamp: new Date().toISOString()
      })
    };
  }
};
```

**Key points:**
- **Prompt Engineering**: We give Claude specific context about PNW hiking
- **Error Handling**: Graceful fallbacks for production use
- **CORS Support**: Enables frontend-backend communication

Create `backend/lambda/package.json`:

```json
{
  "name": "pnw-hiking-chatbot-lambda",
  "version": "1.0.0",
  "dependencies": {
    "@aws-sdk/client-bedrock-runtime": "^3.0.0"
  }
}
```

## Step 3: Infrastructure as Code with AWS CDK

Now let's define our AWS infrastructure. Create `infrastructure/package.json`:

```json
{
  "name": "pnw-hiking-chatbot-infrastructure",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "deploy": "cdk deploy",
    "synth": "cdk synth"
  },
  "devDependencies": {
    "@types/node": "18.14.6",
    "aws-cdk": "2.87.0",
    "typescript": "~4.9.5"
  },
  "dependencies": {
    "aws-cdk-lib": "2.87.0",
    "constructs": "^10.0.0"
  }
}
```

Create the main CDK stack `infrastructure/lib/pnw-hiking-chatbot-stack.ts`:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';
import * as iam from 'aws-cdk-lib/aws-iam';
import { Construct } from 'constructs';

export class PnwHikingChatbotStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 🪣 Private S3 bucket for React app
    const websiteBucket = new s3.Bucket(this, 'WebsiteBucket', {
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });

    // 🌐 CloudFront distribution for global delivery
    const originAccessIdentity = new cloudfront.OriginAccessIdentity(this, 'OriginAccessIdentity');
    
    const distribution = new cloudfront.Distribution(this, 'Distribution', {
      defaultBehavior: {
        origin: new origins.S3Origin(websiteBucket, { originAccessIdentity }),
        viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
      },
      defaultRootObject: 'index.html',
    });

    websiteBucket.grantRead(originAccessIdentity);

    // 🔐 Cognito User Pool for authentication
    const userPool = new cognito.UserPool(this, 'UserPool', {
      selfSignUpEnabled: true,
      signInAliases: { email: true },
      autoVerify: { email: true },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
      },
    });

    const userPoolClient = new cognito.UserPoolClient(this, 'UserPoolClient', {
      userPool,
      generateSecret: false,
    });

    // ⚡ Lambda function for chat processing
    const chatLambda = new lambda.Function(this, 'ChatFunction', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'chat-handler.handler',
      code: lambda.Code.fromAsset('../backend/lambda'),
      timeout: cdk.Duration.seconds(30),
    });

    // 🔑 Grant Lambda permission to call Bedrock
    chatLambda.addToRolePolicy(
      new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        actions: ['bedrock:InvokeModel'],
        resources: [
          `arn:aws:bedrock:${this.region}::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0`,
        ],
      })
    );

    // 🚪 API Gateway for HTTP endpoints
    const api = new apigateway.RestApi(this, 'ChatApi', {
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
        allowHeaders: ['Content-Type', 'Authorization'],
      },
    });

    const chatResource = api.root.addResource('chat');
    chatResource.addMethod('POST', new apigateway.LambdaIntegration(chatLambda));

    // 📤 Output important values
    new cdk.CfnOutput(this, 'WebsiteURL', {
      value: `https://${distribution.distributionDomainName}`,
    });
    
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: api.url,
    });
  }
}
```

Create `infrastructure/bin/app.ts`:

```typescript
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { PnwHikingChatbotStack } from '../lib/pnw-hiking-chatbot-stack';

const app = new cdk.App();
new PnwHikingChatbotStack(app, 'PnwHikingChatbotStack');
```

## Step 4: React Frontend with Chat Interface

Create the React app structure. First, `frontend/package.json`:

```json
{
  "name": "pnw-hiking-chatbot-frontend",
  "version": "0.1.0",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "typescript": "^4.9.5",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  }
}
```

Create the main chat interface `frontend/src/components/ChatInterface.tsx`:

```typescript
import React, { useState, useRef, useEffect } from 'react';

interface Message {
  id: string;
  text: string;
  sender: 'user' | 'bot';
  timestamp: Date;
}

const ChatInterface: React.FC = () => {
  const [messages, setMessages] = useState<Message[]>([
    {
      id: '1',
      text: "Hi! I'm your PNW hiking assistant. Ask me about trails, gear, weather, or safety!",
      sender: 'bot',
      timestamp: new Date()
    }
  ]);
  const [inputText, setInputText] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const sendMessage = async () => {
    if (!inputText.trim() || isLoading) return;

    const userMessage: Message = {
      id: Date.now().toString(),
      text: inputText,
      sender: 'user',
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);
    setInputText('');
    setIsLoading(true);

    try {
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: inputText }),
      });

      const data = await response.json();
      
      const botMessage: Message = {
        id: (Date.now() + 1).toString(),
        text: data.response,
        sender: 'bot',
        timestamp: new Date()
      };

      setMessages(prev => [...prev, botMessage]);
    } catch (error) {
      console.error('Error:', error);
      const errorMessage: Message = {
        id: (Date.now() + 1).toString(),
        text: "Sorry, I'm having trouble right now. Please try again!",
        sender: 'bot',
        timestamp: new Date()
      };
      setMessages(prev => [...prev, errorMessage]);
    }

    setIsLoading(false);
  };

  return (
    <div className="chat-container">
      <div className="messages-container">
        {messages.map((message) => (
          <div key={message.id} className={`message ${message.sender}-message`}>
            <div className="message-content">
              <p>{message.text}</p>
            </div>
          </div>
        ))}
        {isLoading && (
          <div className="message bot-message">
            <div className="message-content">
              <div className="typing-indicator">Thinking...</div>
            </div>
          </div>
        )}
        <div ref={messagesEndRef} />
      </div>
      
      <div className="input-container">
        <input
          value={inputText}
          onChange={(e) => setInputText(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Ask about PNW hiking trails, gear, or safety..."
          disabled={isLoading}
        />
        <button onClick={sendMessage} disabled={!inputText.trim() || isLoading}>
          Send
        </button>
      </div>
    </div>
  );
};

export default ChatInterface;
```

## Step 5: Deployment

Create a deployment script `deploy-website.bat` (Windows):

```batch
@echo off
echo 🚀 Deploying PNW Hiking Chatbot...

echo 📦 Installing dependencies...
cd infrastructure && npm install && cd ..
cd frontend && npm install && cd ..

echo 🏗️ Deploying infrastructure...
cd infrastructure && npx cdk deploy --require-approval never && cd ..

echo 📦 Building React app...
cd frontend && npm run build && cd ..

echo 📤 Uploading to S3...
aws s3 sync frontend/build/ s3://YOUR-BUCKET-NAME --delete

echo ✅ Deployment complete!
```


<img width="669" height="963" alt="image" src="https://github.com/user-attachments/assets/1cdbe13d-f708-428c-bb29-2fffb4cb0031" />

<img width="889" height="926" alt="image" src="https://github.com/user-attachments/assets/88422b0d-6f53-40d1-898f-fb8bd7add3d4" />


## Step 6: Testing Your Chatbot

1. **Deploy the infrastructure:**
   ```bash
   cd infrastructure
   npm install
   npx cdk bootstrap  # First time only
   npx cdk deploy
   ```

2. **Build and upload frontend:**
   ```bash
   cd frontend
   npm install
   npm run build
   # Upload build files to your S3 bucket
   ```

3. **Test with sample questions:**
   - "What are the best beginner trails near Seattle?"
   - "What gear do I need for winter hiking?"
   - "Tell me about bear safety in the Olympics"

## Key Features Explained

### 🎯 Prompt Engineering
The secret sauce is in our prompt design. We give Claude specific context about:
- Geographic focus (PNW)
- Domain expertise (hiking, trails, gear, safety)
- Response style (conversational, practical)
- Length constraints (under 200 words)

### 🔐 Security Best Practices
- Private S3 bucket with CloudFront access only
- Cognito for user authentication
- HTTPS everywhere
- IAM least-privilege permissions

### ⚡ Serverless Benefits
- Pay only for actual usage
- Automatic scaling
- No server management
- Global content delivery

## Cost Considerations

**Typical costs for moderate usage:**
- Lambda: ~$0.20/month (1M requests)
- API Gateway: ~$3.50/month (1M requests)
- Bedrock: ~$3.00/1K input tokens, ~$15.00/1K output tokens
- CloudFront: ~$0.085/GB transferred
- S3: ~$0.023/GB stored

**Total estimated cost: $10-50/month** depending on usage.

## Conclusion

You've just built a production-ready AI chatbot using cutting-edge serverless technology! The combination of Amazon Bedrock's Claude 3.5 Sonnet with AWS's serverless services creates a powerful, scalable, and cost-effective solution.

The key takeaways:
- **Prompt engineering** is crucial for domain-specific AI
- **Serverless architecture** scales automatically and reduces costs
- **Security** should be built-in from the start
- **User experience** matters as much as the AI capabilities

Your PNW hiking chatbot is now ready to help outdoor enthusiasts discover the beautiful trails of Washington and Oregon! 🌲

*Happy hiking and happy coding!* 🥾💻
