# Harness: what we intend to do

Source: `meetings/harness-overview-transcript.txt`. Items marked **(no ticket)** have no ML ticket that I could find on 2026-11-20.

## Decisions and direction agreed in the meeting
- **The harness replaces mt-gorilla.** The new Traci will be built on it.
- **New repo.** Maxi first said to use a branch in mt-gorilla, then settled on a new repo. Deploy to **wip/dev first**. mt-gorilla keeps running in parallel, because SQL Analyst and the AB InBev work depend on it, and is retired later.
- **Release via a front-end feature flag.** Both UIs stay available, and the flag shows or hides the harness-backed UI. Both backends run at the same time, so in-flight demos aren't affected.
- **Agent-first, leaning toward one agent.** Product (Anna, Poppy) wants this direction. Agents differ only by system prompt, MCP servers and skills. Concern to keep in mind (Maxi): the context should come from where the user opened the agent, and users shouldn't have to say "I want to work on X" to get the right skills.
- **Use `deepagents` for this release.** Custom graphs remain an option for special-purpose agents.
- **Design rule:** don't build features the agent can already do when asked. Add a skill for anything repeatable. Add a tool only when it's very specific.
- **Visualization in 4 levels, starting with level 1:**
  1. Basic graphs (port what SQL Analyst does)
  2. Arbitrary code-generated graphs
  3. HTML or interactive components in chat
  4. Dashboards
- **Report format:** the team leans toward self-contained HTML with a Highcharts wrapper, printable to PDF. This was never formally agreed with Anna.
- **This quarter's deliverable:** a Compliance data chat interface as a side panel. This is Q4 roadmap row "AI chat to refine answers" (Must, L, owner Ram, not started). Q4 scope is Phase 1 only, and it's blocked on the per-measurement vs always-on UX decision. See [`plans/q4-2026-ml-roadmap.md`](../../plans/q4-2026-ml-roadmap.md).

## Q4 2026 roadmap items that depend on the harness
- **Traci Harness** (Must, Sagar, in progress). The Notion page is blank; this folder is the detail.
- **AI chat to refine answers**, Compliance (Must, L, Ram, not started)
- **Updated Traci in Insights**, Scope 1, 2 & 3 data (Should, Sagar, not started). Needs the Snowflake MCP.
- **Traci Ingestion Assistant V2** (Should, L, Sagar, not started)

## Sagar's next steps (before a deployment)
1. Fix **queuing** bugs **(no ticket)**. ML-789 is closed, so this probably needs a follow-up.
2. Fix **background job** bugs **(no ticket)**
3. **Graph images in the chat UI** (ML-790, in progress)
4. **Snowflake MCP** (ML-791)

When those four are done, anyone can run it locally and contribute, e.g. to the MCP work.

**Then, for an actual deployment:**
- Port the API integration and auth from mt-gorilla (Maxi: "you just need to port that part") **(no ticket)**
- Deployment workflows and infra for wip **(no ticket)**. Brian usually takes infra tasks.
- Move the code into the new repo and share it **(no ticket)**

**Insights output → Bonobo** (Brian, 2026-11-20): start targeting getting the Insights output from the harness into Bonobo. This is a multi-step effort, beginning with the Sprint 23 spike **ML-812** (Sagar, with Rohit consulted).

**Communication:** Sagar will share a write-up of next steps listing what he needs from front end, Data Ops and back end. He offered to join their planning sessions.

## Asks of other teams
**Front end (Franco)**
- Split the UI into a shared **chat core** plus optional components behind feature flags: threads, side panels, file pane.
- Decide where the UI lives: keep it in the harness repo, move it to an existing front-end repo, or make it a separate package.
- Decide how the front end deploys and toggles between backends.
- Build a generic "agent asks the user" UI: approvals, multi-select, single-select and free text.
- Check UX with Anna. For example, ingestion is project/file-first with the agent on the side, while the demo is chat-first.

**Data Ops (Juan)**
- Maxi asked Juan for a Data Ops ticket to **start on the Snowflake MCP** and its authentication.
- **Fix Traci secret keys.** GPG keys were generated locally and uploaded to Snowflake by hand. Move this to Terraform so private keys are never handled manually and can be rotated. This blocks one of Sagar's tickets. Maxi will take it on.

**Back end**
- Build **MCP servers per platform**, owned by back end: Scope 3 data, and Compliance (measurements, answers, guidance). Work out the auth each one needs. Sagar builds the client side.
- MCP **resources**, e.g. the Snowflake schema and compliance framework knowledge, so the agent can pull context.

**Ram**
- Spike on **RAG architectures for agents** (probably ML-788). Decide between a Compliance MCP and the current workflow. Bulk runs stay as workflows.

**Product (Anna, Poppy)**
- How the CS-team ↔ customer handoff works inside projects: stages, anomaly detection, reusable skills.
- Whether customers can add their own skills.

## Later / ideas
- Skill hierarchy: agent → user → tenant
- **Memory** at user level (e.g. "always answer in JSON") and tenant level (company report tone and structure)
- Customers connect their own Canva / brand kit for reports and PPTs
- React components in chat with interactive graphs
- Desktop app alongside web
- Option to show or hide the agent's steps. Maxi wants both.
- **Guardrails and usage limits**, so it isn't used as a general ChatGPT/Claude replacement
- **Token tracking per tenant**, and per user once the user ID is sent. This might allow usage-based pricing, though clients will likely want a flat rate. Track closely for the first few months against the pricing estimate.
- Sagar's idea of a secrets service with expiry reminders. Maxi prefers Terraform.

## Open questions
- Skill hierarchy and memory scope (user vs tenant): product decision
- Where the shared UI lives, and how the front end is deployed and toggled
- How to structure prompts when most context comes from MCP
- A safety check for harmful commands, like Claude Code's auto mode. Today the sandbox is the only boundary.
- Pricing and cost per tenant for an agent product, since agents use more tokens than workflows
- How custom (non-MCP) tools are added per agent. This is also open in the LCA draft (A1a).
