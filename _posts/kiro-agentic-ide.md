# 👻 Meet KIRO: Your AI-Powered Development Companion 🚀

<img width="851" height="504" alt="image" src="https://github.com/user-attachments/assets/a483eb37-9225-4d1d-930b-3fc9c24f7490" />



Today, I'm excited to introduce you to KIRO, an innovative agentic IDE that's transforming how we build software. What makes KIRO special? It goes beyond simple AI chat assistance, introducing a powerful spec-driven development approach combined with intelligent automation through agent hooks.


### Getting Started 🎯

Getting up and running with KIRO is straightforward:


1. Visit [kiro.dev](https://kiro.dev/) and download the installer
2. Run the installer for your OS (Windows, macOS, or Linux)
3. Launch KIRO and start coding!

<img width="1172" height="597" alt="image" src="https://github.com/user-attachments/assets/d5459ccd-a7ae-45a3-b914-e60a3c78e3b8" />


### First Run Setup

When you first open KIRO, you'll go through a quick setup:

* Choose your login method (social or AWS Builder Id options available)
* Import your VS Code settings and extensions (optional)
* Select your preferred theme
* Set up shell integration for command execution

  <img width="1956" height="1111" alt="image" src="https://github.com/user-attachments/assets/be3d34ed-ccd4-40f6-a518-552f9ba1ccd9" />


### Opening Your Project

Two simple ways to get started:
``bash

Option 1: From terminal

```
kiro .
```
Option 2: From KIRO interface

Open KIRO and select your project or work with Kiro to create a new project from a spec``

### Watch my video: How I used KIRO to create my passion project AI chatbot
[First Look at Kiro Agentic IDE](https://youtu.be/ai65z0DMMow?si=XU5Pw9KHepZq1x_J)

### Core Features 💫

#### Steering Files

Think of steering files as your project's DNA. They're markdown documents that tell KIRO about your:

* Architecture
* Tech stack
* Conventions
* Project structure

To create them:

1. Click the ghost icon in the sidebar
2. Select "Generate Steering Docs"
3. Customize the generated files (product.md, structure.md, tech.md)

Pro tip: Add custom steering files like test-driven-development.md to enforce specific development practices!


#### Vibe Coding Mode

For quick tasks and exploration, use vibe coding through the agentic chat. Perfect for:

* Codebase questions
* Quick prototyping
* Experimental features

### Spec-Driven Development

This is where KIRO really shines. Here's the workflow:


1. Requirements Phase

* Describe your feature
* KIRO generates PM-style requirements
* Review and adjust as needed

1. Design Phase

* Technical design documentation
* TypeScript interfaces
* Architecture plans
* Implementation details

1. Implementation Phase

* Organized task list
* Step-by-step execution of each task that can be further refined from the Kiro chat
* Track progress

### Agent Hooks ⚡

Automate repetitive tasks with event-based triggers. Set up hooks for:

* Auto-documentation updates
* README maintenance
* Design system sync
* Project management integration

To create a hook:

1. Open the KIRO pane (ghost icon)
2. Click + next to "agent hooks"
3. Configure your automation

### MCP Server Integration

Extend KIRO's capabilities by connecting external services. Here's an example Asana integration:

``json
{
  "mcpServers": {
    "asana": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.asana.com/sse"]
    }
  }
}``


### Real-World Impact 🌟

The spec-driven approach has been particularly powerful for building production-ready features with teams.


### Next Steps 🎯

KIRO is currently in public preview with generous free limits. Ready to transform your development workflow?


1. Download KIRO from kiro.dev
2. Set up your first project
3. Create your steering files
4. Start building with specs!


