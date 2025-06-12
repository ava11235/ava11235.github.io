# 🤖 Amazon Q Developer CLI: Your AI Coding Assistant in the Terminal! 🚀

![image](https://github.com/user-attachments/assets/95454bb3-5d5a-4d39-b112-ccf1e928e686)


## 🔍 What is Amazon Q Developer?

Amazon Q Developer is AWS's AI-powered coding assistant that helps developers write code, solve problems, and navigate documentation. It integrates with various development environments including the command line to provide contextual code suggestions and answers to technical questions.

## 🖥️ Amazon Q Developer CLI Features

Amazon Q Developer supercharges your command line experience with powerful AI-driven capabilities:

- **CLI Integration**: Enables completions for hundreds of popular command line tools including git, npm, docker, and aws. As you type, Amazon Q intelligently suggests the most relevant subcommands, options, and arguments.

- **Context-Aware Assistance**: Amazon Q analyzes your current terminal context to provide more relevant suggestions and responses. It understands what you're working on and tailors its help accordingly.

The CLI tool works across multiple platforms and shell environments, making it a versatile addition to any developer's toolkit regardless of their preferred operating system or terminal setup.

## 💻 Installing Amazon Q Developer 🐧

I'll demonstrate how to install on Ubuntu (which I'm running inside WSL in VS Code), but you can find installation instructions for macOS, Linux AppImage, and other environments in the [official AWS documentation](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html)

When using the Amazon Q Developer CLI, you need to authenticate to Q Developer with your Builder ID. After launching, follow the authentication prompts with either Builder ID or IAM Identity Center.

If planning to interact with AWS services, as I will in this demo, you need to configure your AWS credentials by running the  ``` aws configure ```command.
 You can use Q Developer CLI for many tasks that don't require AWS credentials, including writing code like Python scripts.


### Ubuntu Installation Steps: 📦

```bash
# Download the package
wget https://desktop-release.q.us-east-1.amazonaws.com/latest/amazon-q.deb

# Install the package
sudo apt-get install -f
sudo dpkg -i amazon-q.deb

# Launch Amazon Q
q
```

## 💬 Chat with Amazon Q in Your Terminal

The most interactive way to use Amazon Q is through the chat feature:

```bash
q chat
```
The q command by itself is the base command for Amazon Q CLI, 
but to start an interactive chat session like this one, you use the q chat subcommand.

This opens an AI chat session right in your terminal! With properly configured AWS credentials, you can ask questions about your AWS resources. For example:

- "Do I have any EC2 instances running in us-east-1?" 🖥️
- "List my S3 buckets in us-east-1" 🪣

These commands will help you quickly check your AWS resources without having to navigate through the AWS console or use multiple AWS CLI commands.


![image](https://github.com/user-attachments/assets/03be06a7-5dde-4266-9fb2-d1ef64684a69)

To exit the chat, type `/quit`. 

If you want to see all available options and subcommands for Amazon Q CLI, you can run ```/help ``` for more 
information.

![image](https://github.com/user-attachments/assets/b3173b85-703d-4290-83aa-8843180b1ebe)


## 🛠️ Beyond Chat: Other Useful Commands

Amazon Q isn't just for chatting! Here are some other helpful commands:

- `q doctor` - Diagnose and fix installation issues 🩺
- `/save` - Export your conversation to a JSON file 💾
- `/load` - Import a previous conversation 📂
- `/tools` - Manage permissions for tools Amazon Q can use 🔨
- `/context` - Manage context information available to Amazon Q 📋
- `/model` - Select different AI models for your session 🧠

  ![image](https://github.com/user-attachments/assets/366882ae-594f-4a7f-afd3-7c46a5f19e09)


Amazon Q Developer CLI has added a powerful fuzzy search capability for slash commands that you can access with Ctrl + S. This intuitive feature allows you to quickly find and execute commands without needing to remember their exact syntax. When you press Ctrl + S during a Q chat session, a search interface appears where you can type partial command names and see matching slash commands instantly. For example, typing "con" might show both "/context" and "/connect" commands. This fuzzy matching makes the CLI significantly more accessible, especially for new users who are still learning the available commands or experienced developers who use certain specialized commands infrequently. The search results show not only the command names but also brief descriptions of what each command does, helping you select exactly the functionality you need without interrupting your workflow to check documentation.


## 🎉 Conclusion

Amazon Q Developer brings AI assistance directly to your terminal. While I've shown the Ubuntu installation (perfect for WSL users!), remember to check the official documentation for other platforms. Give it a try - your command line will never feel the same again!

Happy coding! 💻✨
