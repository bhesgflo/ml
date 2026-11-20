# Harness: what we have

Source: Sagar's demo in `meetings/harness-overview-transcript.txt`. **Everything here is a local POC on Sagar's machine.** It isn't deployed anywhere, auth only works locally, and Sagar calls it buggy.

## Repo and app layout
- Separate repo from mt-gorilla. It's a monorepo with `apps/` (currently `web`) plus the harness backend.
- The chat UI reuses the same assistant UI library as Forest Elephant. It is still **one UI**, not yet split into a shared chat core plus optional components.
- Slides, architecture diagrams and a draft migration plan exist (mostly generated with Claude).

## Agent core
- Prompt in, response out, with **tool calling** making it an agent.
- Generic system prompt of about 10 lines: general assistant, where to put files, and basic approval rules.
- **One common agent API.** The front end creates a session with an agent name defined in the backend, and the session remembers which agent it is. All agents use the same endpoints, unlike mt-gorilla's per-agent routes (`/data-ingestion/...`, `/api/v1/sql/...`).
- **Per-agent config:** each agent has its own prompt, tools and MCP servers. An agent's `kind` picks the engine. `deep` = LangChain `deepagents`, which has built-in file create/edit and bash tools. Custom LangGraph graphs are still possible.
- **Skills:** loaded from the repo and scoped **per agent**. The name and description go into the system prompt.

## Background jobs and queuing
- The agent can start a background job and keep chatting. When the job finishes, the agent is triggered automatically to act on the output (ML-789).
- MCP background jobs can also be used, so they don't need to be defined in the harness.
- **Message queuing** like Claude Code: messages sent during a run are queued and picked up next. This was needed because of background jobs.
- ⚠️ **Both are buggy.** Sagar called them "a big pain point".

## Sandbox (code execution)
- Runs in **AWS Bedrock AgentCore Code Interpreter**, which has its own CPU, memory, storage and network policy (internet can be off, or ingress limited). mt-gorilla instead runs code as a subprocess of the FastAPI server.
- The agent writes and runs Python and bash and checks its own work.
- Session TTL is currently **1 hour** (max 8 h). Billing is per hour used, and idle time costs less.
- Uploaded and created files are kept in **S3 per session/thread** and copied back into a new sandbox session, so files survive TTL expiry. This could be scoped per project.

## Files and outputs
- File upload, plus an in-app viewer for Excel and other formats. ⚠️ The viewer library is Chrome-only and will probably be replaced.
- **File versioning:** overwriting a file creates a new version.
- Demo: **data ingestion with no domain knowledge.** Given an uploaded file and a template, it wrote and ran the code to map the file into the template structure.
- **Graphs:** made with Python and saved as PNG. There is no UI to show them in chat yet (ML-790).
- **Documents:** docx with tables (python-docx), Excel and CSV. HTML → PDF works on request with nothing extra built.

## Approvals and visibility
- Approvals UI (approve / reject / say something else) behind an approvals flag. It was **off** in the demo.
- The agent's steps and tool calls show in the chat. They can't be hidden yet.
- There is no "auto mode" safety check that catches harmful commands. The sandbox is the safety boundary.

## Observability
- **Langfuse is built in** and traces every tool call, input and output. Use a local Langfuse for development; Langfuse Cloud is **prod only**, because it charges per traced request.
- The existing (non-harness) Langfuse dashboards track cost **per tenant, per service and per model**, not per user, because the user ID isn't sent.
- Rough prod workflow spend from the meeting (garbled; treat as approximate): about $500 Compliance and $300 Scope 3 over roughly the last 2–3 months. Maxi's earlier estimate for the Traci assistant was about $3K/year.

## Data model
- About 90% of tables overlap with mt-gorilla. A few are new for the queue. App-specific data such as data-ingestion "steps" isn't in the common model.

## Not there yet (acknowledged in the meeting)
- MCP support (ML-791)
- In-chat images and graphs (ML-790)
- Hiding the agent's steps
- Skills at user or tenant level, and skills added by customers
- Memory
- Real auth, deployment, and API integration with the platform
- Guardrails and usage limits
- UI split into components
