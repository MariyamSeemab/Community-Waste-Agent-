<div align="center">
 🏆 Agents for Humans Hackathon Submission

Track: Good Neighbor Agents An agent that helps groups of people, not just one: neighborhoods, nonprofits, food banks, schools, libraries, and small local orgs.

# 🤖 Good Neighbor Agent

### Community Waste Recovery — turn surplus into support.

**A grounded AI agent that helps a whole community recover and redistribute surplus food instead of throwing it away.**

[![Live App](https://img.shields.io/badge/Live_App-AWS_Amplify-FF9900?style=for-the-badge&logo=awsamplify&logoColor=white)](https://main.dfwth112aeze8.amplifyapp.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Deployed_end--to--end-brightgreen?style=for-the-badge)](#deployed-resources)

![Strands Agents](https://img.shields.io/badge/Strands_Agents-Python-3776AB?logo=python&logoColor=white)
![Bedrock AgentCore](https://img.shields.io/badge/Bedrock-AgentCore-FF9900?logo=amazonaws&logoColor=white)
![Amazon Nova](https://img.shields.io/badge/Amazon-Nova_Pro-FF9900?logo=amazonaws&logoColor=white)
![MCP](https://img.shields.io/badge/Protocol-MCP-6E56CF)
![AWS Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?logo=awslambda&logoColor=white)
![API Gateway](https://img.shields.io/badge/API_Gateway-FF4F8B?logo=amazonapigateway&logoColor=white)
![Cognito](https://img.shields.io/badge/Amazon_Cognito-DD344C?logo=amazonaws&logoColor=white)
![AWS SAM](https://img.shields.io/badge/AWS_SAM-FF9900?logo=amazonaws&logoColor=white)
![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?logo=amazoncloudwatch&logoColor=white)

[Live App](https://main.dfwth112aeze8.amplifyapp.com/) ·
[Architecture](#architecture) ·
[What It Can Do](#what-it-can-do) ·
[Getting Started](#getting-started) ·
[Live Agent Demo](#live-agent-demo) ·
[Contributors](#contributors)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Highlights](#highlights)
- [Links](#links)
- [Architecture](#architecture)
- [What It Can Do](#what-it-can-do)
- [Tech Stack](#tech-stack)
- [Repository Layout](#repository-layout)
- [Getting Started](#getting-started)
- [Deployed Resources](#deployed-resources)
- [How the Agent Works](#how-the-agent-works)
- [Live Agent Demo](#live-agent-demo)
- [Configuration Reference](#configuration-reference)
- [Observability](#observability)
- [Troubleshooting](#troubleshooting)
- [Teardown](#teardown)
- [Contributors](#contributors)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Every day, good food and useful goods go to waste while food banks, shelters, and
neighbors go without. The problem is rarely a lack of supply. It's a **matching and
logistics** problem: who has surplus, who needs it, and who can move it before it
spoils.

**Good Neighbor Agent** closes that gap. It connects people who **have** surplus
(grocers, restaurants, schools) with people who can **use** it (food banks, shelters,
neighbors), and routes **volunteers and vehicles** to move it.

The agent is a [Strands](https://strandsagents.com/) agent running on the
**Amazon Bedrock AgentCore Runtime** with **Amazon Nova Pro**. It reaches its backend
tools through an **AgentCore Gateway** (MCP), and those tools serve real community data
from **AWS Lambda + API Gateway**. A browser SPA lets people sign in and chat with it.

```mermaid
flowchart LR
    subgraph HAVE["Have surplus"]
        G["🛒 Grocers"]
        R["🍽️ Restaurants"]
        S["🏫 Schools"]
    end

    AGENT{{"🥕 Good Neighbor Agent<br/>matches · grounds · routes"}}

    subgraph NEED["Can use it"]
        FB["🥫 Food banks"]
        SH["🏠 Shelters"]
        N["👥 Neighbors"]
    end

    subgraph MOVE["Move it"]
        V["🙋 Volunteers"]
        VH["🚚 Vehicles"]
    end

    G --> AGENT
    R --> AGENT
    S --> AGENT
    AGENT --> FB
    AGENT --> SH
    AGENT --> N
    AGENT -.->|"assigns"| V
    AGENT -.->|"assigns"| VH
```

> [!NOTE]
> **Status:** deployed and verified end to end in AWS (`us-east-1`). Ask the live agent
> *"What are the current pantry stock levels?"* and it returns a table built from real
> backend data.

---

## Highlights

| | Feature | Why it matters |
|---|---|---|
| 🎯 | **Grounded answers** | The agent answers **only** from live tool data. It never guesses, and it declines when no tool can serve a request. |
| 🔌 | **Real tools over MCP** | Backend tools are exposed through an AgentCore Gateway and backed by AWS Lambda + API Gateway. |
| 🧠 | **Amazon Nova Pro** | Amazon's own model family, so there is no Marketplace subscription or payment-instrument gate. |
| 🔐 | **Secure by design** | Cognito email one-time-code sign-in, IAM (SigV4) inbound auth, OAuth2 outbound auth via AgentCore Identity, and optional per-request tool filtering with Amazon Verified Permissions. |
| 🧪 | **Runs anywhere** | Explore the UI with no AWS, run the real backend locally with no AWS, then deploy step by step. |
| 🔍 | **Observable** | Every layer logs to Amazon CloudWatch, plus a probe script to inspect raw gateway results. |

---

## Links

| Resource | Link |
|----------|------|
| **Live app** | <https://main.dfwth112aeze8.amplifyapp.com/> |
| **Source code** | <https://github.com/MariyamSeemab/Community-Waste-Agent-> |
| **Project writeup** | [`docs/ABOUT.md`](docs/ABOUT.md) — inspiration, build, challenges, testing |
| **Architecture (AWS icons)** | [`docs/architecture-aws-icons.drawio`](docs/architecture-aws-icons.drawio) |
| **Architecture (plain boxes)** | [`docs/architecture.drawio`](docs/architecture.drawio) |

> [!TIP]
> The live app runs the frontend on AWS Amplify. Sign in with an email and the one-time
> code to try it. In **demo mode**, use code `123456`.

---

## Architecture

<img width="1536" height="1024" alt="Good Neighbor Agent architecture diagram" src="https://github.com/user-attachments/assets/34df7e11-8863-4434-829b-435f5efbfcb9" />

*Editable sources: [`docs/architecture-aws-icons.drawio`](docs/architecture-aws-icons.drawio) uses the official AWS service icons (Amplify, Cognito, Bedrock/AgentCore, API Gateway, Lambda, Secrets Manager, IAM, CodeBuild, ECR, CloudWatch). [`docs/architecture.drawio`](docs/architecture.drawio) is a simpler plain-box version of the same flow.*

### System overview

```mermaid
flowchart TB
    U(["👤 Community member"])

    subgraph FE["Frontend · AWS Amplify"]
        SPA["Browser SPA<br/>static/"]
    end

    subgraph AUTH["Identity"]
        COG["Amazon Cognito<br/>email one-time code"]
    end

    subgraph AGENT["AI agent · Amazon Bedrock AgentCore"]
        RT["AgentCore Runtime<br/>Strands agent · agent.py"]
        NOVA["Amazon Nova Pro<br/>via Bedrock"]
        ID["AgentCore Identity<br/>outbound OAuth2 provider"]
        GW["AgentCore Gateway<br/>MCP · openApiSchema target"]
    end

    subgraph BE["Backend · AWS SAM"]
        APIGW["HTTP API Gateway"]
        LAM["AWS Lambda<br/>guidelines · community · pantry · logistics"]
        DATA[("Seed data<br/>JSON")]
    end

    CW["Amazon CloudWatch<br/>logs"]

    U --> SPA
    SPA -->|"sign in"| COG
    COG -->|"JWT"| RT
    SPA -->|"chat"| RT
    SPA -.->|"direct mode"| APIGW
    RT <--> NOVA
    RT -->|"get token"| ID
    RT -->|"MCP tool calls"| GW
    GW -->|"HTTPS"| APIGW
    APIGW --> LAM --> DATA
    RT -.-> CW
    GW -.-> CW
    LAM -.-> CW
```

### Two ways the frontend reaches data

Both paths are built:

```mermaid
flowchart LR
    subgraph P1["Direct · great for demos, no LLM involved"]
        A1["Browser SPA"] -->|"BACKEND_API_URL"| B1["Backend API"]
    end

    subgraph P2["Through the agent · natural language"]
        A2["Browser SPA"] --> R2["AgentCore Runtime<br/>Nova agent"]
        R2 --> G2["AgentCore Gateway"]
        G2 --> B2["Backend API"]
    end
```

- **Direct** — the SPA calls the backend API directly (`BACKEND_API_URL`). Great for
  demos; no agent or LLM is involved.
- **Through the agent** — the SPA calls the AgentCore Runtime. The Nova agent decides
  which tools to call via the Gateway and answers in natural language.

### Request lifecycle

What happens when someone asks *"Which food bank needs the available produce?"*

```mermaid
sequenceDiagram
    autonumber
    actor User as Community member
    participant SPA as Browser SPA
    participant RT as AgentCore Runtime<br/>(Strands agent)
    participant LLM as Amazon Nova Pro
    participant ID as AgentCore Identity
    participant GW as AgentCore Gateway (MCP)
    participant API as API Gateway + Lambda

    User->>SPA: Ask a question
    SPA->>RT: Invoke runtime (Cognito JWT or IAM SigV4)
    RT->>LLM: Prompt + available tools
    LLM-->>RT: Decide which tools to call
    RT->>ID: Get OAuth2 token for the gateway
    ID-->>RT: Access token
    RT->>GW: MCP tool call with bearer token
    GW->>API: HTTP request (openApiSchema target)
    API-->>GW: JSON from real seed data
    GW-->>RT: Tool result
    RT->>LLM: Compose answer from the tool result
    LLM-->>RT: Markdown table + summary
    RT-->>SPA: Grounded response
    SPA-->>User: Rendered answer
```

---

## What It Can Do

The agent answers **only** from live tool data. Its tools:

| Capability | What it provides |
|------------|------------------|
| **Donation & food-safety guidelines** | Acceptance rules and safe donation windows |
| **Surplus donation listings** | Surplus food and goods posted by donors |
| **Community resource catalog** | Categories of items circulating in the network |
| **Recipient needs** | What food banks, shelters, and partners are requesting |
| **Pantry / stock levels** | Current stock at partner pantries, with a low-stock flag |
| **Volunteers & vehicles** | Available drivers and vehicles for pickups and deliveries |

Together these are backed by **seven tools** exposed through the Gateway.

**Example question:**

> *"A grocer has 40 lbs of produce that must be picked up by Friday — which food bank
> needs it, and who could drive it there?"*

### Grounded by design

```mermaid
flowchart TD
    Q(["User question"]) --> AVP{"AVP_POLICY_STORE_ID set?"}
    AVP -->|"Yes"| F["Filter tool groups for the caller<br/>Amazon Verified Permissions"]
    AVP -->|"No"| ALL["Load all registered tool groups"]
    F --> D{"Can a tool serve<br/>this request?"}
    ALL --> D
    D -->|"Yes"| T["Call the tool via the Gateway"]
    T --> R["Present the tool result<br/>Markdown table + summary"]
    D -->|"No"| X["Politely decline<br/>never guess"]
```

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Agent framework** | Strands Agents (Python) |
| **Agent hosting** | Amazon Bedrock AgentCore Runtime |
| **Model** | Amazon Nova Pro (`us.amazon.nova-pro-v1:0`) via Strands `BedrockModel` |
| **Tool access** | AgentCore Gateway (MCP) with an `openApiSchema` target |
| **Identity & auth** | Amazon Cognito (email one-time code), AgentCore Identity (outbound OAuth2), IAM SigV4, optional Amazon Verified Permissions |
| **Backend** | Stdlib-only Python handlers on AWS Lambda + HTTP API Gateway, deployed with AWS SAM |
| **Frontend** | Static SPA (`static/`) on AWS Amplify (or S3 + CloudFront) |
| **Container build** | AWS CodeBuild builds the ARM64 image remotely, so no local Docker |
| **Observability** | Amazon CloudWatch |

---

## Repository Layout

```text
.
├── README.md                     # This file
├── amplify.yml                   # AWS Amplify Hosting build spec (publishes static/)
├── docs/
│   ├── ABOUT.md                  # Project writeup
│   ├── architecture-aws-icons.drawio   # Diagram with official AWS icons
│   └── architecture.drawio       # Editable plain-box architecture diagram
├── schemas/                      # OpenAPI + Lambda schemas (tool contracts)
├── backend/                      # Tool implementations + real seed data
│   ├── README.md                 # Backend structure, data model, run + deploy
│   ├── server.py                 # Local dev server (no AWS) exposing every tool
│   ├── lambda_function.py        # AWS Lambda entrypoint (routes by path)
│   ├── template.yaml             # AWS SAM template (Lambda + HTTP API Gateway)
│   ├── samconfig.toml            # SAM deploy defaults (stack name, region)
│   ├── common.py                 # Shared data loading + response helpers
│   ├── data/                     # Seed JSON: resources, surplus, needs, pantry,
│   │                             #   volunteers, vehicles, guidelines
│   └── handlers/                 # guidelines · community · pantry · logistics
└── static/                       # Frontend SPA (deploy via Amplify or S3 + CloudFront)
    ├── index.html                # Login + chat UI + "How it works" panel
    ├── app.js                    # Cognito sign-in + AgentCore/backend calls + rendering
    ├── styles.css                # Animated chat UI
    ├── config.js                 # Runtime config (Cognito, AgentCore, BACKEND_API_URL)
    ├── README.md                 # Frontend + Amplify deploy docs
    └── AgentCode/                # The deployable agent (NOT published to the web)
        ├── agent.py              # Strands agent + AgentCore Runtime entrypoint (Amazon Nova)
        ├── requirements.txt      # Python dependencies (strands-agents pinned)
        ├── deploy-agent-noauth.ps1       # Deploy the agent with IAM auth (no Cognito)
        ├── iam/                          # Standalone execution role trust + permissions
        ├── gateway/                      # Scripts to wire the Gateway to the backend
        │   ├── backend-openapi.json      # Combined OpenAPI spec for the 7 tools
        │   ├── create_target.py          # Create the openApiSchema gateway target
        │   ├── create_oauth_provider.py  # Create the outbound OAuth2 provider
        │   ├── gateway-workload-policy.json  # IAM the gateway role needs
        │   └── probe_tool.py             # Call the gateway's MCP endpoint directly (debug)
        ├── launchAgent.sh                # Cognito/bootstrap-stack flow (alternative)
        └── deploy-agentcore-runtime.sh   # Cognito configure + launch (used by launchAgent.sh)
```

---

## Getting Started

### Clone the repository

```bash
git clone https://github.com/MariyamSeemab/Community-Waste-Agent-.git
cd Community-Waste-Agent-
```

### Pick your goal

Goals **A → F** increase in scope, and each stands on its own.

```mermaid
flowchart LR
    A["A · See the UI<br/>Python only"] --> B["B · Run the backend locally<br/>no AWS"]
    B --> C["C · Deploy the backend<br/>AWS SAM"]
    C --> D["D · Deploy the frontend<br/>AWS Amplify"]
    C --> E["E · Deploy the agent<br/>AgentCore, IAM auth"]
    E --> F["F · Wire the Gateway<br/>agent answers from live data"]
```

| Goal | What you get | You need |
|------|--------------|----------|
| **A** | Explore the animated UI in demo mode | Python |
| **B** | The real backend running locally | Python (stdlib only, nothing to install) |
| **C** | Live backend on Lambda + HTTP API Gateway | AWS CLI, AWS SAM CLI, configured credentials |
| **D** | Hosted frontend | Git repo + AWS Amplify console |
| **E** | The Nova agent on AgentCore Runtime | `bedrock-agentcore-starter-toolkit`, Bedrock access to Amazon Nova, an execution role |
| **F** | Agent grounded in real backend data via the Gateway | Everything above |

### A. Just see the UI (no AWS)

Only needs Python.

```bash
cd static
python -m http.server 8000
# open http://localhost:8000 - sign in with any email + code 123456
```

Runs in **demo mode** (mocked replies) so you can explore the animated UI and the
"How it works" panel. Motion respects `prefers-reduced-motion`.

### B. Run the real backend locally (no AWS)

The backend is stdlib-only Python, so there is nothing to install.

```bash
# terminal 1 - start the tools API
python backend/server.py                       # http://localhost:8080
curl "http://localhost:8080/pantry"            # real seed data

# then point the SPA at it: in static/config.js set
#   BACKEND_API_URL: 'http://localhost:8080',
# and serve the frontend as in step A.
```

The SPA badge switches to **Live backend data**, and answers come from the real
handlers and seed data. See [`backend/README.md`](backend/README.md) for the data model
and every endpoint.

### C. Deploy the backend to AWS

Needs the **AWS CLI** and **AWS SAM CLI** with credentials configured.

```bash
cd backend
sam build
sam deploy            # creates CloudFormation stack: good-neighbor-backend (us-east-1)
```

Copy the `ApiBaseUrl` output into `static/config.js` as `BACKEND_API_URL`. The frontend
now talks to your live AWS backend (Lambda + HTTP API Gateway).

### D. Deploy the frontend to AWS Amplify

Push this repo to Git, then in the **Amplify console**:
*New app → Host web app → connect the repo → deploy*.

Amplify auto-detects [`amplify.yml`](amplify.yml), which publishes `static/` and prunes
the backend `AgentCode/` from the web root. To inject `config.js` from Amplify
environment variables instead of committing it, see the commented block in
`amplify.yml`.

### E. Deploy the agent to AgentCore (no Cognito)

This runs the Strands agent on **AgentCore Runtime** with **Amazon Nova Pro**, using
**AWS IAM (SigV4)** inbound auth. There is no Cognito and no `bootstrap-stack`.

**Prerequisites**

- `pip install bedrock-agentcore-starter-toolkit` (gives the `agentcore` CLI)
- **Bedrock model access** for Amazon Nova in your region (Nova needs no Marketplace
  subscription, so there is no payment-instrument gate)
- A standalone execution role. The trust and permissions JSON is in
  [`static/AgentCode/iam/`](static/AgentCode/iam)

```powershell
cd static/AgentCode
# Windows: force UTF-8 so the CLI's console output doesn't crash on cp1252
$env:PYTHONUTF8=1; $env:PYTHONIOENCODING="utf-8"
powershell -ExecutionPolicy Bypass -File .\deploy-agent-noauth.ps1
```

The script runs `agentcore configure` (no `--authorizer-config`, which means IAM auth),
then `agentcore deploy`, which builds the ARM64 container remotely via CodeBuild with no
local Docker. Invoke it, passing a `--runtime-user-id` (see below for why):

```powershell
'{"prompt": "hello"}' | Out-File -Encoding ascii payload.json
aws bedrock-agentcore invoke-agent-runtime --region us-east-1 `
  --agent-runtime-arn <RUNTIME_ARN> --runtime-user-id demo-user `
  --payload fileb://payload.json --content-type application/json --accept application/json out.json
```

> [!NOTE]
> There is also a Cognito-based path (`launchAgent.sh` + `deploy-agentcore-runtime.sh`)
> that uses a `bootstrap-stack` for the Cognito pool, S3, and CloudFront. Use the
> no-Cognito path above unless you already have that stack.

### F. Wire the agent to the live backend (Gateway)

This is what makes the agent answer from **real** data instead of only its own
reasoning. The backend REST API is fronted by an **AgentCore Gateway**, exposed to the
agent over MCP.

```bash
# 1. Create the gateway (auto-creates a Cognito authorizer + gateway role)
agentcore gateway create-mcp-gateway --region us-east-1 --name GoodNeighborGateway

# 2. Register the backend as an MCP tool target (edit IDs in the script first)
python static/AgentCode/gateway/create_target.py

# 3. Create the outbound OAuth2 provider the agent uses to call the gateway
python static/AgentCode/gateway/create_oauth_provider.py

# 4. Point the runtime at the gateway and redeploy
cd static/AgentCode
agentcore deploy --auto-update-on-conflict `
  --env COMMUNITY_GATEWAY_URL=<gateway mcp url> `
  --env COMMUNITY_OAUTH_PROVIDER=good-neighbor-gateway-oauth
```

Then invoke with a data question (again, pass `--runtime-user-id`):

```powershell
'{"prompt": "What are the current pantry stock levels?"}' | Out-File -Encoding ascii q.json
aws bedrock-agentcore invoke-agent-runtime --region us-east-1 `
  --agent-runtime-arn <RUNTIME_ARN> --runtime-user-id demo-user `
  --payload fileb://q.json --content-type application/json --accept application/json out.json
```

The agent returns a Markdown table of the live pantry data plus a summary.

**Why `--runtime-user-id`?** The runtime uses IAM (SigV4) inbound auth, so the outbound
machine-to-machine token flow (AgentCore Identity) needs a workload identity, and the
user id supplies it. Without it you get *"Workload access token has not been set."*

**Debugging:** [`gateway/probe_tool.py`](static/AgentCode/gateway/probe_tool.py) calls
the gateway's MCP endpoint directly (token → list tools → call one), so you can see the
exact tool result the agent receives. This is the fastest way to tell an auth failure
apart from a model-behavior issue.

### Deployment pipeline

```mermaid
flowchart LR
    subgraph BACKEND["C · Backend"]
        S1["sam build"] --> S2["sam deploy"]
        S2 --> S3["CloudFormation stack<br/>good-neighbor-backend"]
        S3 --> S4["Lambda + HTTP API"]
    end

    subgraph FRONTEND["D · Frontend"]
        G1["git push"] --> G2["Amplify build<br/>amplify.yml"]
        G2 --> G3["Published static/"]
    end

    subgraph AGENTDEP["E · Agent"]
        A1["agentcore configure"] --> A2["agentcore deploy"]
        A2 --> A3["CodeBuild<br/>ARM64 container"]
        A3 --> A4["AgentCore Runtime"]
    end

    subgraph GATEWAYDEP["F · Gateway"]
        W1["create-mcp-gateway"] --> W2["create_target.py"]
        W2 --> W3["create_oauth_provider.py"]
        W3 --> W4["Redeploy with env vars"]
    end
```

---

## Deployed Resources

A reference deployment in `us-east-1`. Replace these with the identifiers from your own
deploy.

| Resource | Identifier |
|----------|------------|
| Backend stack | `good-neighbor-backend` (Lambda + HTTP API) |
| Backend API URL | `https://h1aly0x3a1.execute-api.us-east-1.amazonaws.com` |
| Agent runtime | `good_neighbor_agent-m0NZWbGrv5` (Amazon Nova Pro, IAM auth) |
| Runtime exec role | `good-neighbor-agent-exec-role` |
| Gateway | `goodneighborgateway-y0bvoowruo` |
| Gateway target | `BackendTools` (openApiSchema → backend API) |
| Gateway exec role | `AgentCoreGatewayExecutionRole` |
| Outbound OAuth2 provider | `good-neighbor-gateway-oauth` |

---

## How the Agent Works

`agent.py` defines a Strands `Agent` fronted by the AgentCore Runtime
(`BedrockAgentCoreApp`).

- **Model** — Amazon Nova Pro (`us.amazon.nova-pro-v1:0`) via the Strands
  `BedrockModel`. It is Amazon's own model family, so no Marketplace subscription is
  needed. That avoids the `INVALID_PAYMENT_INSTRUMENT` gate that third-party models can
  hit.
- **Tool groups** — the agent registers a tool group only when its gateway URL is
  configured, so it runs with any subset of tools. Nothing connects at import time, and
  connections open per request. In the current deploy the **community** group is wired
  to the Gateway, and that one Gateway exposes all backend tools.
- **Grounded** — the system prompt leads with "use the tool result." When a tool returns
  data the agent must present it (Markdown table + summary), and it declines only when no
  tool can serve the request.
- **Optional AVP** — if `AVP_POLICY_STORE_ID` is set, tools are filtered per request
  against the caller's identity via Amazon Verified Permissions. Unset, all registered
  groups load.

### Auth chain

```mermaid
flowchart LR
    C["Caller<br/>IAM SigV4 + runtime-user-id"] --> R["AgentCore Runtime"]
    R -->|"workload identity"| I["AgentCore Identity"]
    I -->|"OAuth2 access token<br/>good-neighbor-gateway-oauth"| R
    R -->|"Bearer token"| G["AgentCore Gateway<br/>Cognito authorizer"]
    G -->|"gateway execution role"| T["Target: BackendTools<br/>openApiSchema"]
    T --> B["Backend API"]
```

---

## Live Agent Demo

With the deployed AWS environment, invoke the AgentCore runtime using
`--runtime-user-id demo-user`, then try these questions.

**"Show current surplus donations."**

<img width="900" alt="Agent response listing current surplus donations" src="https://github.com/user-attachments/assets/dff0116f-1614-4141-88b0-6119caabbf03" />

**"Which volunteers are available?"**

<img width="900" alt="Agent response listing available volunteers" src="https://github.com/user-attachments/assets/d6c1e9f1-e0c3-452b-b1e7-e9ec1b843182" />

**"What are the current pantry stock levels?"**

<img width="900" alt="Agent response with a table of pantry stock levels" src="https://github.com/user-attachments/assets/b4bae9b6-4fe6-4bcd-b3bb-d6cfdaa8fdd0" />

**"Which food bank needs the available produce?"**

<img width="900" alt="Agent response matching produce to the food bank that needs it" src="https://github.com/user-attachments/assets/b57e9298-2258-482c-847d-c5543e6c60b3" />

### Grounding test

Ask an unrelated question such as **"What's the weather tomorrow?"** The agent should
**decline**, because it only answers from verified tool data.

<img width="900" alt="Agent declining an unrelated weather question" src="https://github.com/user-attachments/assets/0d48f51c-8b21-4472-b1fc-4a2bd3831995" />

---

## Configuration Reference

### Agent runtime

Set environment variables via `agentcore deploy --env KEY=VALUE`:

| Variable | Purpose |
|----------|---------|
| `AWS_REGION` | AWS region (default `us-east-1`) |
| `COMMUNITY_GATEWAY_URL` | MCP URL of the gateway the agent calls |
| `COMMUNITY_OAUTH_PROVIDER` | AgentCore Identity provider name for the gateway |
| `GUIDELINES_GATEWAY_URL` | *(optional)* separate guidelines gateway (SigV4) |
| `PANTRY_GATEWAY_URL` / `PANTRY_OAUTH_PROVIDER` | *(optional)* separate pantry gateway |
| `LOGISTICS_GATEWAY_URL` / `LOGISTICS_OAUTH_PROVIDER` | *(optional)* separate logistics gateway |
| `AVP_POLICY_STORE_ID` | *(optional)* enables per-request AVP tool filtering |

### Frontend

`static/config.js` (`window.WORKSHOP_CONFIG`):

| Key | Purpose |
|-----|---------|
| `BACKEND_API_URL` | Deployed backend API. The SPA answers directly from it |
| `COGNITO_USER_POOL_ID` / `COGNITO_CLIENT_ID` / `COGNITO_REGION` | Cognito email one-time-code sign-in |
| `AGENTCORE_RUNTIME_ARN` / `AGENTCORE_ENDPOINT` | Call the agent runtime from the browser |

### IAM permissions

The wiring requires the following. Each missing permission otherwise surfaces as an
`AccessDeniedException`.

| Role | Required permissions |
|------|----------------------|
| **Runtime role** (`good-neighbor-agent-exec-role`) | Bedrock invoke, `bedrock-agentcore:GetResourceOauth2Token`, `GetWorkloadAccessToken*`, and `secretsmanager:GetSecretValue` on `bedrock-agentcore-identity!default/*` |
| **Gateway role** (`AgentCoreGatewayExecutionRole`) | `bedrock-agentcore:GetWorkloadAccessToken` + `GetResourceApiKey` — see [`gateway-workload-policy.json`](static/AgentCode/gateway/gateway-workload-policy.json) |

---

## Observability

Every layer logs to **Amazon CloudWatch**.

| Component | CloudWatch log group |
|-----------|----------------------|
| AgentCore Runtime | `/aws/bedrock-agentcore/runtimes/good_neighbor_agent-<id>-DEFAULT` |
| AgentCore Gateway | `/aws/vendedlogs/bedrock-agentcore/gateway/APPLICATION_LOGS/goodneighborgateway-<id>` |
| Backend Lambda | `/aws/lambda/good-neighbor-backend` |
| Container build | CodeBuild build logs (per `agentcore deploy`) |

Tail the agent live while invoking it:

```bash
aws logs tail /aws/bedrock-agentcore/runtimes/good_neighbor_agent-<id>-DEFAULT \
  --region us-east-1 --follow
```

The runtime log shows tool registration, each tool call, the model response, and full
tracebacks. This is where the auth-chain `AccessDenied` errors surfaced. The gateway log
is where a tool call can show HTTP 200 while the body is an *"unable to fetch outbound
api key"* error (a missing gateway permission).

Tracing is off by default: the runtime is deployed with `--disable-otel`, and the
gateway's X-Ray trace delivery is left disabled, because logs are enough to operate it.
Enabling OpenTelemetry / X-Ray is a one-flag change for deeper request tracing.

> [!TIP]
> To inspect a tool result without the model, run
> [`gateway/probe_tool.py`](static/AgentCode/gateway/probe_tool.py). Logs tell you
> *where* a request broke, and the probe tells you *what* the gateway returned.

---

## Troubleshooting

```mermaid
flowchart TD
    S(["Agent declines a data question"]) --> P["Run gateway/probe_tool.py"]
    P --> R{"What does the raw<br/>tool result show?"}
    R -->|"Real data"| M["Model or prompt behavior<br/>review the system prompt"]
    R -->|"unable to fetch outbound api key"| I["Gateway role is missing<br/>GetWorkloadAccessToken and GetResourceApiKey"]
    R -->|"Token or auth error"| A["Check the OAuth2 provider<br/>and the gateway authorizer"]
```

| Symptom | Cause and fix |
|---------|---------------|
| **Agent replies "I'm not able to help…" for data questions** | The tool returned an error, or the prompt is over-declining. Run `gateway/probe_tool.py` to see the raw tool result. If it shows *"unable to fetch outbound api key"*, the gateway role is missing `GetWorkloadAccessToken` / `GetResourceApiKey`. |
| **"Workload access token has not been set"** | Invoke with `--runtime-user-id`. |
| **`INVALID_PAYMENT_INSTRUMENT`** | You're on a third-party model. Switch to Amazon Nova, or add a payment method and enable the model in Bedrock. |
| **`agentcore` CLI crashes with a `UnicodeEncodeError` on Windows** | Set `$env:PYTHONUTF8=1; $env:PYTHONIOENCODING="utf-8"` first. |
| **`agentcore deploy` seems to hang or stops mid-build** | Let it run to completion, since it monitors CodeBuild. Use `--auto-update-on-conflict` to update an existing runtime. |
| **SAM CLI shows a non-zero exit on Windows** | Its progress banner goes to stderr. Trust the textual `Successfully created/updated stack` result. |

---

## Teardown

Remove everything a full deploy created (stops any charges):

```bash
# Backend (Lambda + API Gateway)
sam delete --stack-name good-neighbor-backend --region us-east-1

# Agent runtime
agentcore destroy

# Gateway + target, OAuth2 provider, and the IAM roles/Cognito pool the gateway
# auto-created are removed via the console or the bedrock-agentcore-control API.
```

---

## Contributors

Good Neighbor Agent is built by:

<table>
  <tr>
    <td align="center" width="200">
      <a href="https://github.com/MariyamSeemab">
        <img src="https://github.com/MariyamSeemab.png?size=100" width="100" alt="MariyamSeemab" /><br />
        <sub><b>MariyamSeemab</b></sub>
      </a>
    </td>
    <td align="center" width="200">
      <a href="https://github.com/dineshrajdhanapathyDD">
        <img src="https://github.com/dineshrajdhanapathyDD.png?size=100" width="100" alt="dineshrajdhanapathyDD" /><br />
        <sub><b>dineshrajdhanapathyDD</b></sub>
      </a>
    </td>
  </tr>
</table>

---

## Contributing

Contributions, issues, and ideas are welcome.

```mermaid
gitGraph
    commit id: "main"
    branch "feat/new-tool"
    checkout "feat/new-tool"
    commit id: "feat: add tool"
    commit id: "docs: update schema"
    checkout main
    merge "feat/new-tool" id: "PR merged"
    branch "fix/gateway-policy"
    checkout "fix/gateway-policy"
    commit id: "fix: gateway IAM"
    checkout main
    merge "fix/gateway-policy" id: "PR merged "
```

1. **Fork** the repository, then clone your fork and create a branch from `main`:
   ```bash
   git clone https://github.com/<your-username>/Community-Waste-Agent-.git
   cd Community-Waste-Agent-
   git checkout -b feat/your-feature
   ```
2. **Make your changes.** The backend runs locally with no AWS (goal B above), so verify
   affected endpoints with `curl` before opening a PR.
3. **Commit** using [Conventional Commits](https://www.conventionalcommits.org):

   | Prefix | Use for |
   |--------|---------|
   | `feat:` | A new feature |
   | `fix:` | A bug fix |
   | `docs:` | Documentation only |
   | `refactor:` | Code change with no behavior change |
   | `chore:` | Tooling, config, dependencies |

4. **Push** your branch and open a **Pull Request** describing *what* changed and *why*.

**Branch naming:** `feat/…` · `fix/…` · `docs/…` · `refactor/…` · `chore/…`

### Adding a new tool

As a guide, a new tool touches these places:

1. Add seed data in `backend/data/` and a handler in `backend/handlers/`.
2. Route it in `backend/lambda_function.py` and expose it in `backend/server.py`.
3. Update the OpenAPI contracts in `schemas/` and
   `static/AgentCode/gateway/backend-openapi.json`.
4. Update the gateway target so the agent can discover the new tool.

> [!IMPORTANT]
> Never commit real credentials. `static/config.js` holds runtime configuration, so use
> Amplify environment variables (see the commented block in `amplify.yml`) for anything
> environment-specific. Demo code `123456` works in demo mode only.

---

## License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE).

Copyright © 2026 [Mariyam Seemab](https://github.com/MariyamSeemab) and [DineshRaj Dhanapathy](https://github.com/dineshrajdhanapathyDD).

---

<div align="center">

**🤖 Good Neighbor Agent** · *Surplus shouldn't go to waste. Neighbors shouldn't go without.*

[⬆ Back to top](#-good-neighbor-agent)

</div>
