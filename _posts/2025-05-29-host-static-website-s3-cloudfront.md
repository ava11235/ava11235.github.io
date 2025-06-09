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

## 🚀 Implementation Guide: S3 + CloudFront with Origin Access Control

### Step 1: Create Your S3 Bucket and Upload Website Files 📂
```

1. Sign in to the AWS Management Console and open the Amazon S3 console.
2. Choose **Create bucket** ➕.
3. Enter a unique bucket name (e.g., `my-static-website-bucket`).
4. Select your preferred AWS Region 🗺️.
5. Keep all default settings and choose **Create bucket**.
6. Select your new bucket and upload your website files (HTML, CSS, JavaScript, images, etc.) 📄.
```

> 💡 **Note**: Unlike traditional S3 website hosting, you don't need to enable static website hosting on the bucket. We'll be using the REST API endpoint instead of the website endpoint.

### Step 2: Create a CloudFront Web Distribution with Origin Access Control ☁️

```
1. Open the CloudFront console.
2. Choose **Create Distribution** ➕.
3. Under **Origin Domain**, enter or select your S3 bucket 🪣.
4. Under **Origin access**, select **Origin access control settings (recommended)** ✅.
5. In the dropdown list, choose **Create control setting** ⚙️.
6. Name your control setting (e.g., `s3-website-oac`) and leave the default **Sign requests (recommended)** setting enabled.
7. Choose **Create** 🆕.
8. Configure additional distribution settings:
   - **Viewer protocol policy**: Choose **Redirect HTTP to HTTPS** for better security 🔒.
   - **Default root object**: Enter your main page (typically `index.html`).
   - If using a custom domain, enter it under **Alternate domain names (CNAMEs)** 🏷️.
   - For **SSL certificate**, select **Custom SSL Certificate** and request or choose your certificate if using a custom domain 📜.
9. Complete the remaining settings as needed for your specific use case and choose **Create distribution** 🚀.
```

### Step 3: Update Your S3 Bucket Policy 📝

After creating your distribution, CloudFront provides the policy statement needed to give OAC permission to access your S3 bucket:

```
1. Copy the policy provided by CloudFront 📋.
2. Go to your S3 bucket in the AWS console.
3. Go to the **Permissions** tab 🔑.
4. Under **Bucket policy**, choose **Edit** and paste the policy.
5. Choose **Save changes** 💾.
```
The policy will look similar to this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-static-website-bucket/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EDFDVBD6EXAMPLE"
                }
            }
        }
    ]
}
```

### Step 4: Update DNS Records for Your Custom Domain (If Applicable) 🔄

```
1. In your DNS provider's console, create a CNAME record pointing your domain to your CloudFront distribution URL (e.g., `d1234abcd.cloudfront.net`) 🌐.
2. Wait for DNS changes to propagate (can take up to 48 hours, but often much less) ⏳.
```

## 🔄 How I Maintain My Portfolio Site:

1. 📝 Updating Content:

- When I make changes, I upload new files to S3
- For immediate updates, I create a CloudFront invalidation. Why? CloudFront caches content at edge locations to improve performance, but this means updated files might not be immediately visible to users until the cache expires naturally. Invalidation forces an immediate update. It's not strictly necessary if you can wait for natural cache expiration, but it's useful when you need immediate content updates to be visible to all users.

```
aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
```
- Usually just targeting the specific files I changed


2. 📊 Monitoring:

- I occasionally check CloudFront metrics in CloudWatch
- This helps me see visitor patterns and ensure everything's running smoothly


💰 Important: Before we dive into the implementation, you should be aware that AWS is a pay-as-you-go service with costs that vary based on usage.

## 🎯 Conclusion:
- This approach is perfect for static websites like portfolios
- The global distribution means my site loads quickly for everyone
- It's easy to update and maintain
- The default CloudFront URL works great until I decide to add a custom domain
This is how I'm currently hosting my portfolio site using S3 and CloudFront with the default distribution URL. It's been a great solution - my site loads quickly, is always available, and costs very little to maintain.

✅ What I've Learned:

I hope this guide helps you set up your own portfolio website the same way. Feel free to reach out if you have any questions about how I did this!

🌟 Happy Hosting! 🌟
