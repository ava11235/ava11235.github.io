UPDATED: As of June 2025 Amazon CloudFront just got a major upgrade with a simplified onboarding experience that lets developers secure and accelerate their web applications in seconds! I have updated my tutorial accordingly.

# 🚀 How I'm Hosting My Portfolio Website on AWS S3 with CloudFront

🌟 I'm currently hosting my [portfolio website](https://d2b2q92b8w3i9s.cloudfront.net/portfolio.html) using Amazon S3 and CloudFront. I wanted a reliable, fast, and cost-effective solution, and this setup has been perfect. In this guide, I'll walk you through exactly how I did it, using the default CloudFront distribution URL.


![static-website](https://github.com/user-attachments/assets/103084c3-aad8-468b-a7e3-2d64d107cab7)


## 🤔 What is a Static Website?
A static website consists of fixed content that displays the same information for every user. Unlike dynamic websites that generate content on-demand using databases and server-side processing, static sites use pre-built HTML, CSS, and JavaScript files that are delivered directly to the browser without any server-side processing.

Static websites are perfect for portfolios, landing pages, documentation sites, and blogs because they are:
- Lightning-fast (no processing time)
- Easy to deploy and maintain
- Highly reliable

📋 Why I Chose This Setup:
- Reliable hosting with 99.99% uptime
- Global content delivery through CloudFront's edge locations
- Highly secure, since you don't need to make your S3 bucket public

🔍 Here's How I Set Everything Up:

## 🚀 UPDATED Implementation Guide: S3 + CloudFront with Origin Access Control


### 📋 Prerequisites:

AWS account
Website files ready to deploy
Domain name (optional)

### 🔧 Step 1: Create S3 Bucket

- Open AWS Console and navigate to S3

- Click "Create bucket"

- Choose a unique bucket name

- Accept default settings

- Create bucket

![image](https://github.com/user-attachments/assets/3a5f5059-5b67-4632-a9f7-1b6ffde85f18)


### 📦 Step 2: Upload Content

- Upload your website files to S3
- Ensure index.html is in root

![image](https://github.com/user-attachments/assets/e675ba2b-a19c-4259-8f36-e66fea7a54b7)


### ☁️ Step 3: Configure CloudFront

- Navigate to CloudFront console

- Click "Create Distribution"

- Select "Single website or app"

- Enter your domain name (optional)

- Choose "Amazon S3" as origin

- Select your bucket

- Keep "Grant CloudFront access to origin" enabled (default)

### Step 4: Review & Create

![image](https://github.com/user-attachments/assets/97a1092b-75b3-4d65-96d8-ac3890ea10c0)


 Review settings
- Review the Origin and Behaviors settings and customize if needed (not required)

- Create distribution

- Wait for deployment (several minutes usually)

Use CloudFront URL to access your site

- From the General Tab, copy the distribution domain name

- Append /index.html to it. 

- You should be able to access your website!


![image](https://github.com/user-attachments/assets/e6f1ebfc-4dc3-469c-b50b-f07fe22e04af)


💡 Pro Tip: The new simplified process automatically:

• Configures bucket permissions (my favorite part of the update!)
• Sets up origin access control 

• Handles security settings


## 🔄 How I Maintain My Portfolio Site:

1. 📝 Updating Content:

- When I make changes, I upload new files to S3
- For immediate updates, I create a CloudFront invalidation. Why? CloudFront caches content at edge locations to improve performance, but this means updated files might not be immediately visible to users until the cache expires naturally. Invalidation forces an immediate update. It's not strictly necessary if you can wait for natural cache expiration, but it's useful when you need immediate content updates to be visible to all users.

```
aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
```
- Usually just targeting the specific files I changed


2. 📊 Metrics
 
I use the View Metrics in CloudFront

![image](https://github.com/user-attachments/assets/51193b1a-9cb5-48e9-b987-979b4591b275)


- This helps me see visitor patterns and ensure everything's running smoothly


💰 Important: Before we dive into the implementation, you should be aware that AWS is a pay-as-you-go service with costs that vary based on usage.

## 🎯 Conclusion:
- This approach is perfect for static websites like portfolios
- The global distribution means my site loads quickly for everyone
- It's easy to update and maintain
- The default CloudFront URL works great until I decide to add a custom domain
This is how I'm currently hosting my portfolio site using S3 and CloudFront with the default distribution URL. It's been a great solution - my site loads quickly, is always available, and costs very little to maintain.  Next, I plan to add a custom domain. I will blog about it!

✅ What I've Learned:

I hope this guide helps you set up your own portfolio website the same way. Feel free to reach out if you have any questions about how I did this!

🌟 Happy Hosting! 🌟
