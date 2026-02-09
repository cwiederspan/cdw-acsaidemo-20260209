# Copilot Instructions

## Non Negotiables

- **DO NOT store sensitive information** — API keys, secrets, passwords, certificates, and connection strings must NEVER be stored in clear text in source code, configuration files, or anywhere that would be exposed via Git. Use secure alternatives such as environment variables, `dotnet user-secrets`, Azure Key Vault, GitHub Secrets, or equivalent mechanisms for your stack.
- **DO update this file** — Continually improve these instructions as the project evolves. Add project-specific context, architecture decisions, and implementation details so that Copilot (and future contributors) can be maximally effective.
- **DO include best practices and lessons learned** — Document what works, what doesn't, and why. This file should be a living record that grows smarter with every iteration.
- **DO validate changes** — Always run existing linters, builds, and tests before and after making changes. Never introduce regressions.
- **DO make minimal changes** — Prefer surgical, targeted edits over sweeping rewrites. Change only what is necessary to achieve the goal.

## Security Requirements

- **Authentication** — Prefer managed identities and `DefaultAzureCredential` over API keys wherever possible. Enterprise tenants may enforce `disableLocalAuth=true` on Azure resources, making API keys unavailable.
- **Secrets management** — Use `dotnet user-secrets` for local development, Azure Key Vault for deployed environments, and GitHub Secrets for CI/CD pipelines.
- **`.gitignore`** — Ensure sensitive files (`appsettings.Development.json`, `*.pfx`, `.env`, `secrets.json`) are excluded from source control.

## Nice to Haves

- Update command line tools (Azure CLI, .NET SDK, Node.js, etc.) before starting work on a new project.
- Ensure you have the latest version of GitHub Copilot CLI installed.
- When reproducing samples or quickstarts, check NuGet/npm package versions — samples often use outdated or pre-release versions with breaking API changes in newer releases.
- Prefer using ecosystem tools (`dotnet new`, `npm init`, `az` CLI, etc.) over manual file creation to reduce errors.
- Comment code only where clarification is needed — avoid obvious or redundant comments.

## Azure-Specific Guidance

- **Resource naming** — Use a consistent naming convention (e.g., `{prefix}-{service}-{date}`) for easy identification and cleanup.
- **Resource groups** — Create a dedicated resource group per project/experiment for clean teardown.
- **RBAC over keys** — Assign roles to managed identities and user principals rather than using shared API keys.
- **Managed Identity** — Enable System Assigned Managed Identity on resources that need to call other Azure services.
- **Azure CLI extensions** — Some Azure services require CLI extensions that prompt for installation. Run `az config set extension.use_dynamic_install=yes_without_prompt` to avoid interactive prompts during automation.

## Project Setup Conventions

- **Configuration** — Use `appsettings.json` for non-sensitive defaults and structure. Provide real values via environment variables or `dotnet user-secrets`.
- **Build & run commands** — Document the exact commands to restore, build, and run the project in the sections below.
- **Architecture** — Include a brief architecture diagram or description so that Copilot understands the data flow.

---

## Project Overview

### What It Does

This application is an **AI-powered voice agent** built on Azure Communication Services (ACS) Call Automation and Azure OpenAI. When a caller dials the ACS phone number:

1. **EventGrid** delivers an `IncomingCall` webhook to the app
2. The app **answers the call** via the ACS Call Automation SDK
3. A **greeting is played** using Azure Cognitive Services text-to-speech (TTS)
4. The app **listens for speech** using Cognitive Services speech-to-text (STT)
5. Recognized speech is sent to **Azure OpenAI** which returns:
   - A conversational response
   - A sentiment score (1–10)
   - Intent classification and category
6. If sentiment is **low (<5)**, the call is **transferred to a human agent**
7. Otherwise, the AI response is played back and the conversation continues
8. On **silence timeout** or goodbye, the call is hung up

**Source sample**: [Azure-Samples/communication-services-dotnet-quickstarts — callautomation-openai-sample-csharp](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/callautomation-openai-sample-csharp)

### Architecture Flow

```
Caller ──► ACS Phone Number ──► EventGrid (IncomingCall)
                                       │
                                       ▼
                              .NET 8 Minimal API (this app)
                              ┌────────────────────────────┐
                              │  /api/incomingCall         │
                              │  Answer call + attach      │
                              │  event processors          │
                              │                            │
                              │  Cognitive Services        │
                              │  ├─ TTS (play prompts)     │
                              │  └─ STT (recognize speech) │
                              │                            │
                              │  Azure OpenAI              │
                              │  ├─ Chat completions       │
                              │  ├─ Sentiment analysis     │
                              │  └─ Intent detection       │
                              │                            │
                              │  Transfer to agent (PSTN)  │
                              └────────────────────────────┘
```

### Technology Stack

| Component | Technology |
|---|---|
| **Runtime** | .NET 8 (minimal API) |
| **Call Automation** | `Azure.Communication.CallAutomation` NuGet package |
| **Speech** | Azure AI Services (Cognitive Services) — STT/TTS integrated via ACS |
| **AI** | Azure OpenAI — Chat Completions API |
| **Events** | Azure Event Grid (IncomingCall webhook) |
| **Tunneling** | Azure Dev Tunnels (for local development) |
| **Identity** | `Azure.Communication.Identity` for ACS user provisioning |

### Azure Resources Required

| Resource | Purpose | Setup Docs |
|---|---|---|
| **Resource Group** | Container for all resources | [Create RG](https://learn.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal) |
| **Azure Communication Services** | Call Automation, phone number, EventGrid | [Create ACS](https://learn.microsoft.com/azure/communication-services/quickstarts/create-communication-resource) |
| **Phone Number** (via ACS) | Calling-enabled PSTN number | [Get Phone Number](https://learn.microsoft.com/azure/communication-services/quickstarts/telephony/get-phone-number) |
| **Azure AI Services** (multi-service) | Speech-to-text / text-to-speech for calls | [Create AI Services](https://learn.microsoft.com/azure/ai-services/multi-service-resource) |
| **Azure OpenAI** | GPT model for conversational AI | [Create OpenAI Resource](https://learn.microsoft.com/azure/ai-services/openai/how-to/create-resource) |
| **Managed Identity link** | ACS → Cognitive Services trust | [ACS + Cognitive Services Integration](https://learn.microsoft.com/azure/communication-services/concepts/call-automation/azure-communication-services-azure-cognitive-services-integration) |

### Configuration Values

All sensitive values must be stored via `dotnet user-secrets` (local) or Azure Key Vault (deployed). The `appsettings.json` file contains only placeholder keys.

| Key | Description |
|---|---|
| `DevTunnelUri` | Public URL of the Azure Dev Tunnel (e.g., `https://xxxxx.devtunnels.ms`) |
| `CognitiveServiceEndpoint` | Azure AI Services endpoint URL |
| `AcsConnectionString` | ACS resource connection string |
| `AzureOpenAIServiceKey` | Azure OpenAI API key |
| `AzureOpenAIServiceEndpoint` | Azure OpenAI endpoint URL |
| `AzureOpenAIDeploymentModelName` | Deployed model name (e.g., `gpt-4o`) |
| `AgentPhoneNumber` | PSTN number to transfer calls to for human escalation (optional) |

### Build & Run

```bash
# Navigate to the project directory
cd CallAutomationOpenAI

# Restore NuGet packages
dotnet restore

# Build the project
dotnet build

# Run the application (listens on port 5165 HTTP / 7156 HTTPS)
dotnet run
```

### Dev Tunnel Setup

```bash
# Install dev tunnels CLI (if not already installed)
# https://learn.microsoft.com/azure/developer/dev-tunnels/get-started

# Create and host a dev tunnel
devtunnel create --allow-anonymous
devtunnel port create -p 5165
devtunnel host
```

### EventGrid Webhook Registration

After the app and dev tunnel are running, register an EventGrid subscription on the ACS resource for `Microsoft.Communication.IncomingCall` events, pointing to `{DevTunnelUri}/api/incomingCall`. See [Incoming Call Notification docs](https://learn.microsoft.com/azure/communication-services/concepts/call-automation/incoming-call-notification).

---

## Lessons Learned & Best Practices

### NuGet Package Versions

- **CRITICAL**: The original sample uses `Azure.AI.OpenAI 1.0.0-beta.5` (from May 2023). This is extremely outdated. The OpenAI SDK has had **breaking API changes** in later versions (namespace changes, client initialization, chat completions API shape). Always check the latest stable version and adapt code accordingly.
- The `Azure.Communication.CallAutomation` package should be the latest stable (1.1.0+ or newer).
- The `Azure.Communication.Identity` package in the sample is a beta (`1.0.0-beta.4`); prefer the latest stable release.

### Managed Identity Configuration

- ACS must have a **System Assigned Managed Identity** enabled.
- That identity must be granted the **Cognitive Services User** role on the Azure AI Services resource.
- Without this, ACS Call Automation cannot invoke speech services during calls.

### Dev Tunnel Gotchas

- The dev tunnel must be configured for **anonymous access** (`--allow-anonymous`) because EventGrid cannot authenticate to private tunnels.
- The `Helper.cs` file in the sample contains a hardcoded path (`/Users/anuj/bin`) for the devtunnel CLI — this is developer-specific and should be removed or replaced with proper PATH resolution. We manage tunnels separately via the CLI.
- The CliWrap-based tunnel launch in Helper.cs is optional; running `devtunnel host` in a separate terminal is simpler and more reliable.

### EventGrid Webhook Validation

- When registering the EventGrid subscription, Azure sends a `SubscriptionValidation` event. The app must respond with the `validationCode` to complete the handshake. The sample handles this in the `/api/incomingCall` endpoint.
- If the webhook URL is unreachable or the validation fails, the EventGrid subscription won't activate.

### Common Pitfalls

| Issue | Resolution |
|---|---|
| `Azure.AI.OpenAI` API breaking changes | Pin to a known working version or adapt to latest SDK patterns |
| Dev tunnel not accessible | Ensure tunnel is running and port matches app's HTTP port (5165) |
| EventGrid webhook validation fails | App must be running and reachable before registering the subscription |
| ACS can't use Cognitive Services | Enable Managed Identity on ACS and assign Cognitive Services User role |
| Speech recognition fails silently | Check that `CognitiveServiceEndpoint` is correct and the AI Services resource is in a supported region |
| Call transfers fail | Verify `AgentPhoneNumber` is a valid E.164 format number and ACS has outbound calling enabled |
| `maxTimeout` counter is static | The sample uses a shared `var maxTimeout = 2` — this won't work correctly for concurrent calls. Consider per-call state management for production. |

### Success Markers

- [ ] All Azure resources provisioned and linked
- [ ] Project builds with `dotnet build` — no errors
- [ ] Dev tunnel is running and publicly accessible
- [ ] EventGrid subscription is active (validation succeeded)
- [ ] App is running on localhost:5165
- [ ] Inbound call to ACS number triggers the `/api/incomingCall` endpoint
- [ ] AI greeting is played to the caller
- [ ] Speech recognition captures caller's words
- [ ] Azure OpenAI returns a meaningful response
- [ ] Response is played back to the caller via TTS
- [ ] Low-sentiment calls trigger agent transfer flow
- [ ] Call ends gracefully on goodbye or timeout
