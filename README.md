# GitHub Copilot CLI

## Bootstrap Prompt

### Step 1: Setup the Instructions

 I'm going to ask you to access a GitHub repo that contains many samples. I'll pick one of those samples and I'm going to ask you to help read in the instructions from that sample, follow all the directions, and then help me to get it up and running. In this case, the sample is about using Azure, Azure OpenAI, and Azure Communication Services to create a working application that works over voice calling. The repo is located at https://github.com/Azure-Samples and the specific sample within that repo that I want to repoduce is https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/callautomation-openai-sample-csharp. As a first step, I need you to read through that information, help devise a plan, and then persist that information into the copilot-instructions.md file in a way that's works best for you. Also, I want to make sure that you update the file with lessons learned, best practices, success and failure insight, etc. as we go, so that it's ever evolving as we learn the best way to do this. Let's get started.

### Step 2: Launch GitHub Copilot

```bash

# It is good to do this right away
az login -t 16b3c013-d300-468d-ac64-7eda0820b6d3

# Launch GitHub Copilot CLI
copilot --yolo

# Log into GitHub
> /login

# Initialize the session with prompt from above
> /init

# Add plugins if desired

# Microsoft Work IQ
> /plugin marketplace add github/copilot-plugins
> /plugin install workiq@copilot-plugins 

# Microsoft Learn Docs
> /plugin marketplace add microsoftdocs/mcp
> /plugin install microsoft-docs@microsoft-docs-marketplace

```

### Step 3: Possible MCP Servers

- **Azure MCP** - `npx -y @azure/mcp@latest server start`
- **Microsoft Fabric** - `npx -y @microsoft/fabric-mcp@latest server start --mode all   `
- **Playwright** - `npx -y @playwright/mcp@latest`
- **Azure DevOps** - `npx -y @azure-devops/mcp your-org-name`
