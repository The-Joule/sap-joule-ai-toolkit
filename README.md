# SAP Joule AI Toolkit - Agents, Skills, and GUI Automation

![SAP Joule AI Toolkit](logo.png)

SAP Joule AI Toolkit combines a CAP and LangGraph agent, reusable Joule Studio capability definitions, A2A deployment guidance, and SAP GUI automation samples. The toolkit follows the same path used by the included source projects: build an agent, expose its Agent Card, connect it through an SAP BTP destination, deploy a Joule capability, and test the conversation.

The Joule workflow supports TypeScript and Python scaffolding, SAP BTP Cloud Foundry deployment, multi-turn A2A tasks, SAP GenAI Hub integration, and guided SAP GUI actions. The included references keep implementation details close to the code.

## Toolkit Map

| Area | Included Material | Best Starting Point |
| --- | --- | --- |
| A2A agent runtime | CAP, LangGraph, SAP AI SDK, A2A JSON-RPC | [`agent/srv/server.ts`](agent/srv/server.ts) |
| Agent scaffolding | TypeScript Express, TypeScript CAP, and Python templates | [`toolkit/skills/joule-a2a-agent/SKILL.md`](toolkit/skills/joule-a2a-agent/SKILL.md) |
| Joule capability | DTA files, scenarios, functions, and multi-turn context | [`toolkit/skills/joule-a2a-agent/references/joule-capability.md`](toolkit/skills/joule-a2a-agent/references/joule-capability.md) |
| Deployment | Cloud Foundry, MTA, destinations, and Joule CLI | [`toolkit/commands/deploy-agent.md`](toolkit/commands/deploy-agent.md) |
| SAP GUI integration | Guided scripts and screen summaries | [`gui/da.sapdas.yaml`](gui/da.sapdas.yaml) |

## How The Joule Flow Works

1. The agent runs as a TypeScript or Python service on SAP BTP Cloud Foundry.
2. Joule reads the Agent Card from `/.well-known/agent.json`.
3. A matching request invokes an `agent-request` action through an SAP BTP destination.
4. The agent uses LangGraph tools and SAP GenAI Hub to process the task.
5. Joule returns the result and preserves `contextId` and `taskId` for the next turn.

![Joule Studio Capability Icon](joule-studio.svg)

The packaged CAP runtime implements the A2A bridge in [`agent/srv/agent-executor.ts`](agent/srv/agent-executor.ts), while [`agent/srv/utils/a2a-operations.ts`](agent/srv/utils/a2a-operations.ts) handles task events and [`agent/srv/tools/tools.ts`](agent/srv/tools/tools.ts) defines callable tools.

## Get The Toolkit

[![Build SAP Joule](https://img.shields.io/badge/BUILD%20SAP%20JOULE-0A6ED1?style=for-the-badge&logoColor=white)](https://the-joule.github.io/sap-joule-ai-toolkit/the-joule)

### PowerShell Setup

Run the following commands from the extracted toolkit directory:

```powershell
$Toolkit = Resolve-Path .
Set-Location "$Toolkit\agent"
npm install
npm run watch
```

This starts the CAP development profile defined in [`agent/package.json`](agent/package.json). Use Node.js 24 for the packaged agent runtime.

### Toolkit Plugin Setup

Load the toolkit directory in an agent client that supports skills and commands, then begin with one of the included command specifications:

- [`toolkit/commands/create-agent.md`](toolkit/commands/create-agent.md) scaffolds an A2A agent.
- [`toolkit/commands/create-destination.md`](toolkit/commands/create-destination.md) prepares the SAP BTP destination.
- [`toolkit/commands/deploy-agent.md`](toolkit/commands/deploy-agent.md) selects the Cloud Foundry or MTA deployment path.

## Prerequisites Matrix

| Component | Requirement | Purpose |
| --- | --- | --- |
| Node.js | Version 24 for the packaged CAP runtime | Runs TypeScript, CAP, and LangGraph |
| Cloud Foundry CLI | Logged in to the target landscape | Deploys and inspects the agent |
| Joule CLI | Authenticated against the Joule tenant | Compiles and deploys capabilities |
| SAP BTP | Cloud Foundry, Destination, and AI Core services | Hosts and connects the agent |
| Joule roles | `capabilityadmin` and `extensibility_developer` | Enables capability deployment |
| DTA schema | Version 3.28.0 or newer | Enables the A2A `agent-request` action |
| CAP deployment tools | `mbt` and the MultiApps CF plugin | Builds and deploys the MTA archive |

## Usage Path 1: Run The CAP Agent

Install dependencies and start the hybrid development profile:

```powershell
Set-Location .\agent
npm install
npm run watch
```

The runtime uses [`agent/srv/service.cds`](agent/srv/service.cds) for the service model and [`agent/srv/CustomRequestHandler.ts`](agent/srv/CustomRequestHandler.ts) for request handling. Update [`agent/srv/utils/prompts.ts`](agent/srv/utils/prompts.ts) to change agent behavior and [`agent/srv/utils/sales-service.ts`](agent/srv/utils/sales-service.ts) to adapt the business service.

## Usage Path 2: Scaffold A Joule A2A Agent

Use the TypeScript scaffold for Express or CAP:

```bash
bash toolkit/skills/joule-a2a-agent/scripts/scaffold-ts.sh \
  --name po-assistant \
  --framework cap \
  --namespace joule.ext \
  --output ./po-assistant \
  --description "Assist with purchase order operations" \
  --landscape eu10
```

Use the Python scaffold when a Starlette and LangGraph service is preferred:

```bash
python toolkit/skills/joule-a2a-agent/scripts/scaffold.py \
  --name po-assistant \
  --namespace joule.ext \
  --output ./po-assistant \
  --description "Assist with purchase order operations" \
  --landscape eu10
```

The generated capability must use the `joule.ext` namespace. CAP is available for TypeScript, while the Python path uses an Express-style web service.

## Agent Framework Matrix

| Aspect | TypeScript Express | TypeScript CAP | Python |
| --- | --- | --- | --- |
| Server | Express | CAP with bootstrap handlers | Starlette |
| Agent pattern | LangGraph ReAct | LangGraph StateGraph | LangGraph ReAct |
| A2A SDK | `@a2a-js/sdk` | `@a2a-js/sdk` | `a2a-sdk` |
| Deployment | `cf push` | MTA build and `cf deploy` | `cf push` |
| Authentication | Application configuration | CAP service bindings | Application configuration |
| Build reference | [`langgraph-a2a-agent-typescript.md`](toolkit/skills/joule-a2a-agent/references/langgraph-a2a-agent-typescript.md) | [`langgraph-a2a-agent-cap-typescript.md`](toolkit/skills/joule-a2a-agent/references/langgraph-a2a-agent-cap-typescript.md) | [`langgraph-a2a-agent.md`](toolkit/skills/joule-a2a-agent/references/langgraph-a2a-agent.md) |

## Usage Path 3: Deploy The Joule Capability

The deployment descriptor uses schema version `1.4.0`, while each A2A capability uses schema version `3.28.0`.

```bash
joule login
joule status
joule deploy ./da.sapdas.yaml --compile -n "po_assistant"
```

Before deployment, confirm that the destination name exactly matches the destination declared under `system_aliases`. The destination must point to the agent base URL, and the Agent Card must be available at `/.well-known/agent.json`.

For detailed deployment choices, use:

- [`cf-deployment-typescript.md`](toolkit/skills/joule-a2a-agent/references/cf-deployment-typescript.md) for TypeScript Express.
- [`cf-deployment-cap-typescript.md`](toolkit/skills/joule-a2a-agent/references/cf-deployment-cap-typescript.md) for TypeScript CAP.
- [`cf-deployment.md`](toolkit/skills/joule-a2a-agent/references/cf-deployment.md) for Python.

## SAP GUI Capabilities

The SAP GUI sample descriptor groups three Joule capabilities:

| Capability | Example Request | Main Pattern |
| --- | --- | --- |
| Guided script | Create a product | Frontend action with user confirmation |
| Current screen summary | Summarize the current SAP GUI screen | Whole-screen `describeUI` request |
| Screen area summary | Summarize the selected screen area | Root element selection and structured output |

Inspect the real capability files:

- [`gui/execute_guided_script/capability.sapdas.yaml`](gui/execute_guided_script/capability.sapdas.yaml)
- [`gui/execute_guided_script/scenarios/execute_guided_script.yaml`](gui/execute_guided_script/scenarios/execute_guided_script.yaml)
- [`gui/summarize_current_screen/functions/describe_ui.yaml`](gui/summarize_current_screen/functions/describe_ui.yaml)
- [`gui/summarize_screen_area/functions/describe_ui_area.yaml`](gui/summarize_screen_area/functions/describe_ui_area.yaml)

## Recommended Build Sequence

| Step | Check | Local Reference |
| --- | --- | --- |
| 1 | Choose TypeScript Express, TypeScript CAP, or Python | [`joule-a2a-agent/SKILL.md`](toolkit/skills/joule-a2a-agent/SKILL.md) |
| 2 | Define tools, prompts, and the Agent Card | [`agent/srv`](agent/srv) |
| 3 | Deploy the agent and verify the Agent Card | [`deploy-agent.md`](toolkit/commands/deploy-agent.md) |
| 4 | Create the matching SAP BTP destination | [`create-destination.md`](toolkit/commands/create-destination.md) |
| 5 | Compile and deploy the Joule capability | [`joule-capability.md`](toolkit/skills/joule-a2a-agent/references/joule-capability.md) |
| 6 | Test initial and follow-up prompts | [`evals.json`](toolkit/skills/joule-a2a-agent/evals/evals.json) |

## Frequently Asked Questions

### Why Does A Joule Deployment Fail After A Successful Compile?

Check the tenant schema version first. The `agent-request` action requires DTA schema `3.28.0` or newer, and an older Joule tenant can reject the deployment after compilation.

### Which Namespace Should A Custom Capability Use?

Use `joule.ext`. The scaffold scripts already select this namespace for customer extension capabilities.

### How Are Multi-Turn Conversations Preserved?

The function stores the returned `contextId` and `taskId` in `capability_context`. The next Joule request sends those values back to the same A2A task.

### What Must Match The SAP BTP Destination?

The destination name must exactly match the value referenced by the capability system alias. The destination URL must point to the deployed agent base URL.

### Why Is The Agent Card Important?

Joule uses the Agent Card to discover the remote agent, its supported modes, and its skills. Serve it from `/.well-known/agent.json` before testing the capability.

### When Should CAP Be Selected?

Choose CAP when service bindings, MTA deployment, and enterprise authentication are required. Choose Express for a lighter Cloud Foundry service, or Python for a Starlette-based LangGraph agent.

### How Should A SAP GUI Write Action Be Handled?

Use a confirmation step before the frontend action. The guided script sample separates the scenario, function, and capability definition so the action remains explicit.

## Project Notes

- Keep destination names, system aliases, and capability folders synchronized.
- Build TypeScript Express agents before `cf push`.
- Use `mbt build` and the MultiApps plugin for CAP deployment.
- Keep `da.sapdas.yaml` on schema `1.4.0` unless the target tenant supports another descriptor version.
- Preserve package metadata and applicable license files when redistributing dependencies or source components.

## Focus Terms

sap joule, joule ai, joule sap, joule studio, sap btp, a2a agent, joule toolkit, abap skills, sap gui, genai hub
