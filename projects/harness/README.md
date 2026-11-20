# Harness

The general-purpose agent harness Sagar is building. It is meant to **replace mt-gorilla** and become the base for every ML agent (Traci, SQL Analyst, Compliance, LCA, …). Owner: **Sagar Thacker**.

| File | What's in it |
|---|---|
| [current-state.md](current-state.md) | What exists today (demoed), plus known bugs and gaps |
| [roadmap.md](roadmap.md) | What we intend to do: decisions, next steps, asks of other teams, open questions |
| [../../plans/q4-2026-ml-roadmap.md](../../plans/q4-2026-ml-roadmap.md) | Q4 2026 roadmap ML rows, four of which depend on the harness |
| [meetings/](meetings/) | Source material. `harness-overview-transcript.txt` is Sagar's demo + architecture walkthrough, transcribed automatically (date not in the file; added to this repo 2026-11-20) |

## Glossary (the transcript garbles these)

| Heard in transcript | Means |
|---|---|
| Mountain Gorilla / Montgarrita / Martin Garrila | `mt-gorilla`, the current agent backend |
| Forest Elephant / "A elephant" | front-end app whose assistant UI the harness reuses |
| Panda / Red Panda | the harness project (repo, UI library and backend). "orangutan red panda" was said about where the UI could live. Confirm repo names with Franco |
| Tracy | Traci |
| Bono | Bonobo |
| LAN fuse / Lancers | Langfuse |
| Analyst / ABI / "ABM Bev" | SQL Analyst; AB InBev work and demo |
| code interpreter | AWS Bedrock AgentCore Code Interpreter (the sandbox) |
| deep agents | LangChain `deepagents` |

## Related Jira tickets

| Key | Summary | Status (2026-11-20) |
|---|---|---|
| ML-607, ML-611 | [SPIKE] Agent Harness (Sagar, Ram) | Closed |
| ML-714 | [Tech-Design] Agent Harness | Closed |
| ML-789 | [harness] Incorporate Background Jobs for an Agent | Closed, but the meeting says background jobs and queuing are still buggy |
| ML-790 | [harness] Showcase images/graphs/svg in Chat | In Progress |
| ML-791 | [harness] Add MCP support for Snowflake | To Do |
| ML-792 | Migrate SQL Analyst as an agent to the new harness | To Do |
| ML-803 | [harness] Answer every pending tool call when a HIL approval is auto-cancelled | To Do (Sprint 23) |
| ML-788 | Complyrag RAG architecture spike (Ram). Likely the "RAG for agents" spike mentioned in the meeting | — |
| A1a / A1b in `drafts/sprint-23.md` | LCA agent on the harness | Draft |
