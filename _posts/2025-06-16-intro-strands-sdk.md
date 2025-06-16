# 🚀 Getting Started with Strands SDK: Building Your First AI Agent

As AI development becomes more accessible, frameworks like Strands SDK are making it easier than ever to build capable AI agents. In this post, we'll explore how to create an agent that accesses AWS documentation and generates architecture diagrams using Amazon Q Developer CLI to assist us. ✨


## 🤔 What is Strands SDK?

Strands SDK is a lightweight, code-first framework for building AI agents. It provides a simple but powerful way to create agents that can leverage different language models and tools while remaining production-ready.

Key features include:

* 🪶 Simple agent creation and customization
* 🔄 Support for multiple model providers
* 🧰 Built-in tools and capabilities
* 👥 Multi-agent and autonomous agent support
* 🔒 Focus on safety and security

🔌 Understanding Model Context Protocol (MCP)

Model Context Protocol (MCP) is an open protocol that standardizes how applications provide context to Large Language Models (LLMs). This is crucial for our example because:


* 🧩 MCP enables Strands agents to access external tools and services
* 🌐 It creates a standardized way for models to communicate with specialized servers
* 🧰 MCP servers provide additional functionality without requiring custom code

Important note: When working with MCP tools in Strands, all agent operations must be performed within the MCP client's context manager (using a with statement). This ensures the MCP session remains active while the agent is using the tools.

```python
# Correct way to use MCP tools
with mcp_client:
    agent = Agent(tools=mcp_client.list_tools_sync())
    response = agent("Your prompt here")
```


## ⚙️ Setting Up Your Environment

First, let's set up our development environment:

In this example I used VS Code with Ubuntu WSL. There are plenty of other great IDEs out here to choose from.

1. In your IDE's terminal, install the Strands SDK and required packages:

```bash
pip install strands-agents uv sarif-om
```


🧩 Make sure Amazon Q Developer CLI is available in your IDE. If you are new to Q Developer CLI, check out my [blog](https://ava11235.github.io/2025/06/11/q-developer-cli.html) post on this topic.

## 🛠️ Creating an AWS Documentation & Diagram Agent

Let's create an agent that can access AWS documentation and create architecture diagrams. In VS Code, we can use Amazon Q Developer CLI to help us:

```bash
"Help me write a Strands agent that uses MCP to access AWS documentation and create architecture diagrams"
```

After getting guidance from Amazon Q Developer CLI, create a file named aws_docs_diagram_agent.py with the following code:

```python
from mcp import StdioServerParameters, stdio_client
from strands import Agent
from strands.models import BedrockModel
from strands.tools.mcp import MCPClient

# Create client for AWS documentation MCP server
aws_docs_client = MCPClient(
    lambda: stdio_client(
        StdioServerParameters(
            command="uvx", args=["awslabs.aws-documentation-mcp-server@latest"]
        )
    )
)

# Create client for AWS diagram MCP server
aws_diag_client = MCPClient(
    lambda: stdio_client(
        StdioServerParameters(
            command="uvx", args=["awslabs.aws-diagram-mcp-server@latest"]
        )
    )
)

# Configure Bedrock model
bedrock_model = BedrockModel(
    model_id="us.anthropic.claude-3-5-haiku-20241022-v1:0",
    temperature=0.7,
)

# Define system prompt for the agent
SYSTEM_PROMPT = """
You are an expert AWS Solutions Architect with comprehensive knowledge of AWS services, architectures, and best practices. Your purpose is to assist customers in:

1. Understanding AWS architectural best practices aligned with the Well-Architected Framework
2. Designing optimal cloud solutions for their specific use cases
3. Resolving technical challenges in existing AWS implementations
4. Making informed decisions about AWS service selection and configuration

Capabilities:
- You can search and reference official AWS documentation to provide accurate, up-to-date information
- You can generate clear AWS architecture diagrams when requested
- When creating diagrams, always specify the complete file path so customers can easily locate them

Response Guidelines:
- Begin responses with a concise summary of your recommendation
- Provide sufficient technical detail while remaining accessible to varying technical backgrounds
- Reference specific AWS documentation when applicable
- Consider cost, performance, security, reliability, and operational excellence in your recommendations
- If information is insufficient to make a recommendation, ask clarifying questions

Maintain a professional, helpful tone while providing technically accurate guidance.
"""

# Initialize agent with MCP tools and model - MUST use with statement for MCP context
with aws_diag_client, aws_docs_client:  # Using context managers ensures MCP connections stay active
    all_tools = aws_docs_client.list_tools_sync() + aws_diag_client.list_tools_sync()
    agent = Agent(tools=all_tools, model=bedrock_model, system_prompt=SYSTEM_PROMPT)

    # Run the agent with a sample prompt
    response = agent(
        "Get the documentation for hosting a static website on S3 and CloudFront and then create a diagram of the services including a client desktop accessing the website"
    )
```


## ▶️ Running Your Agent

To run the agent:
```bash
python aws_docs_diagram_agent.py
```

The agent will:

1. 📚 Fetch AWS documentation on hosting static websites with S3 and CloudFront
2. 🖼️ Generate an architecture diagram showing how these services connect
3. 🖥️ Include a client desktop accessing the website in the diagram
4. 📝 Provide a detailed explanation along with the diagram

Here is a [video](https://www.youtube.com/watch?v=pqEypzSw58c&t=70s0) of how I used Q Developer CLI and Strands SDK to explain and work with the above code.

🔍 How MCP Enhances Agent Capabilities

In our example, MCP enables our agent to:

* 📘 Access and search official AWS documentation through the aws-documentation-mcp-server
* 🎨 Generate professional architecture diagrams via the aws-diagram-mcp-server
* 🔄 Combine these capabilities seamlessly without writing custom integration code

The real power comes from combining multiple MCP servers to create agents with diverse capabilities. You can add additional MCP servers for cost estimation, security analysis, and more!


## ✅ How Amazon Q Developer CLI Enhances Development

Amazon Q Developer CLI can help at various stages:


* 🔍 Find examples:  "Show me examples of using MCP clients in Strands"
* 🐛 Debug issues: "Explain error in MCPClient initialization"
* 💡 Optimize code:  "How can I make my Strands agent more efficient?"
* 📚 AWS documentation: "How do I set up CloudFront with S3?"

## 🔄 Customizing Your Agent

Need to adjust your agent? Ask Amazon Q Developer CLI:

```bash
 "Show me how to add another MCP server to my Strands agent"
```

Or to get help with MCP context managers:

```bash
"Explain best practices for working with multiple MCP clients in Strands"
```


## 🔮 Next Steps

From here, you can:

1. 🧩 Add more MCP servers for different capabilities
2. 🔄 Create interactive agents that can have conversations about architecture
3. 📊 Generate reports with diagrams and cost estimates
4. 🌐 Extend to multi-cloud architecture planning

## 🏁 Conclusion

Strands SDK combined with MCP tools and Amazon Q Developer CLI creates a powerful platform for building AWS architecture assistants. The standardized MCP protocol allows your agent to leverage specialized servers for documentation and diagrams while maintaining a clean, simple codebase. 🌟

For more information:

* 📘 Strands SDK Documentation
* 🤖 Amazon Q Developer Documentation
* ☁️ AWS MCP Servers Repository
* 🔌 MCP Protocol Specification

This example demonstrates just one possibility with Strands SDK - the combination of multiple specialized MCP servers opens up endless possibilities for creating powerful, knowledgeable agents! 🚀🤖✨
