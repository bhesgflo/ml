# Sprint 23 — Draft tickets (Q4 gaps)

**Sprint:** Sprint 23 (id 5841), Nov 23 – Dec 7, 2026. Brian said on 2026-11-20 to target Sprint 23 for now. Tickets blocked on other work (H2→R1, S1→S2, R3) can carry over. The **⇡ S23** marks are left over from when these were Sprint 24 candidates.
**Status:** all created in Sprint 23 on 2026-11-20, to be polished in sprint planning: H1 = ML-821, H2 = ML-809, H3 = ML-810, R1 = ML-822, R2 = ML-825, R3 = ML-823, S1 = ML-811, S2 = ML-816.
**Source:** gaps found in `projects/harness/roadmap.md` (meeting next steps with no ticket) and `plans/q4-2026-ml-roadmap.md` (Q4 roadmap rows with no ML ticket).

Legend: ✅ approve as-is · ✏️ edit · ❌ drop

---

## H. Harness: path to a wip deployment

These come before the roadmap items, because the Compliance chat (R1) and Traci in Insights (R3) need a deployed harness. Order: H1 → H2 / H3 in parallel → R1.

### H1. ✅ **CREATED ML-821** — [harness] Make queued messages and background-job wake-ups reliable ⇡ S23
- **Type:** Story · **Labels:** `Platform`, `enhancement` · **Assignee:** Sagar Thacker
- **Follows:** ML-789, which is Closed, but its "Still open" question was never answered. In the harness overview meeting Sagar called queuing and background jobs "a big pain point" and his first two fixes before a deployment.
- ✏️ Sagar to add the specific bugs he's seen. Split into two tickets if they're unrelated.

## Problem

Messages sent during a run are queued, and a finished background job starts a wake-up run. Both work in the demo but are buggy. ML-789 left open what happens when a job finishes while a run is active or a new message arrives.

## Goal

Queued user messages and job wake-ups run one at a time per thread, in a defined order, with nothing lost or run twice.

## Acceptance criteria

* While a run is active, new user messages queue and run in order after it.
* A job that finishes during an active run queues its wake-up run behind it, instead of starting a second concurrent run.
* A wake-up and a user message arriving together are both handled, in a documented order.
* Each queued item runs exactly once, even if the API restarts.
* The user sees queued items and job status in the thread.
* Tests cover: job finishes mid-run, message arrives mid-wake-up, and job fails.

## Still open

* The specific bugs from Sagar's testing (✏️).

---

### H2. ✅ **CREATED ML-809** — Deploy the harness (panda) to wip
- **Type:** Task · **Labels:** `Platform` · **Assignee:** Brian Hirst (confirmed 2026-11-20), with Sagar Thacker
- **Context from the meeting:** new repo, deployed to dev first, with mt-gorilla running alongside it.

**Context**
The harness only runs on Sagar's laptop. The front end can't integrate with it, and nobody else can test it, until it runs in wip.

**Scope**
* The panda repo is on GitHub with CI, and the team can run it locally (the meeting said after H1, ML-790 and ML-791).
* Infra for wip: API service, Postgres (including the new queue tables), the S3 bucket for session files, and IAM for the Bedrock AgentCore Code Interpreter and Bedrock models.
* Code Interpreter network policy: decide what the sandbox can reach. The default is no internet.
* Secrets come from AWS Secrets Manager. Nothing is committed to the repo.
* Langfuse: runs are traced to a self-hosted or local Langfuse in wip. Langfuse Cloud stays prod only.
* Deploy workflow: `develop` → wip.

**Acceptance criteria**
* The harness API runs in wip, and the demo agent completes a chat, a sandbox code run and a background job end to end.
* Sandbox files persist to S3 and come back after the sandbox TTL expires.
* Runs show up in Langfuse.
* The Terraform is merged. mt-gorilla in wip is unaffected.

---

### H3. ✅ **CREATED ML-810** — [harness] Port platform auth and the agent API integration from mt-gorilla
- **Type:** Story · **Labels:** `Platform`, `enhancement` · **Assignee:** Sagar Thacker
- **Context from the meeting:** Maxi said this is mostly porting from mt-gorilla. Franco plans a front-end feature flag that switches between the mt-gorilla and harness backends.

## Problem

Auth in the harness only works locally. Front-end apps can't call it the way they call mt-gorilla, so the front end has nothing to point its feature flag at.

## Goal

A front-end app authenticates against the harness the same way it does against mt-gorilla, and uses the common agent API: create a session with an agent name, then stream.

## Acceptance criteria

* Requests use the platform's existing auth. Tenant and user come from the token, not from the client.
* Tenant ID **and user ID** are attached to Langfuse traces, which adds the per-user cost tracking the meeting said is missing.
* Threads, files and sandbox sessions are isolated per tenant. One tenant can't read another's.
* The API contract (session create, message, stream, approvals, files) is documented for the front end.
* Works in wip behind H2.

## Still open

* Which DB tables carry over from mt-gorilla (about 90% overlap) and which stay mt-gorilla-only.

---

## R. Q4 roadmap rows with no ML ticket

### R1. ✅ **CREATED ML-822** — [harness][Complyrag] Compliance chat to refine AI answers (Phase 1)
- **Type:** Story · **Labels:** `Compliance`, `enhancement` · **Assignee:** Ram Gandikota (confirmed 2026-11-20), with Sagar Thacker supporting on the harness side
- **Roadmap:** "AI chat to refine answers": Must Have, size L, not started. Customer: Hirschmann Auto, who offered to help.
- **Blocked by:** the UX decision (below), H2 and H3. It also depends on what ML-788 recommends.

## Problem

AI Suggested Response gives one answer from the trained documents, and users can't refine it. CS and customers say this is the biggest blocker, so users copy answers into Claude instead.

## Goal

From a compliance question, a user can chat with a harness agent to refine the AI answer, using only documents already trained into Tracera.

## Acceptance criteria

* The user can ask follow-ups on an answer: make it shorter, change the tone, search the documents again, or leave a named document out.
* Answers draw only on that tenant's trained documents, filtered by fiscal year (from the ML-786 follow-up).
* Answers are in the company's voice, consistent with ML-787.
* Answers cite the documents they used.
* The agent runs in wip through the harness, and runs are traced in Langfuse with tenant and user.
* The API is ready for the front-end side panel. The panel itself is FE work.

## Out of scope

* Phase 2 from the roadmap: past-year responses, cross-framework answers, Data Links via MCP.
* Bulk "prepare with Traci", and report writing.

## Still open / Blocked by

* **UX decision:** chat per measurement or always on? Poppy and Colby are designing the flow. Sagar needs the answer to build the agent.
* How the agent reaches the trained documents: a Compliance MCP server or a complyrag tool. ML-788's recommendation should decide.
* The roadmap page says timing is still to be confirmed with Engineering, and its notes are pending Sagar's sign-off.

---

### R2. ✅ **CREATED ML-825** — [Abbyy] Migrate Barnes Industrial to the Abbyy IWG tenant
- **Type:** Story · **Labels:** `Site_Based`, `enhancement` · **Assignee:** Chethan Patharla
- **Roadmap:** "Migrate Barnes Industrial to New Abbyy Process": Must Have, size M, **In progress** in Notion.
- ✏️ Chethan isn't in `TICKET_CONVENTIONS.md`. Check whether work is already tracked elsewhere, e.g. under epic ML-342 Abbyy 2.0. Look up Chethan's account ID before creating. The details below are a placeholder until Chethan fills them in.

**Goal:** Barnes Industrial bills go through the new Abbyy process on the IWG tenant, which improves extraction accuracy and cuts manual checking.

**Scope:**
* Migrate Barnes's existing files to the Abbyy IWG tenant.
* Route new Barnes bills to the new tenant.
* Compare extraction on a sample of Barnes bills, old process vs new, before cutover.
* Check whether ML-206 (the Usage Format bug on Barnes prod) still applies after the migration, and close or fix it.

**Acceptance criteria:**
* All new Barnes bills are processed on the IWG tenant.
* Extraction results on the sample match or beat the old process, with any differences listed.
* The old route is switched off for Barnes.

---

### R3. ✅ **CREATED ML-823** — [harness][Traci] Traci in Insights over Scope 1, 2 and 3 data
- **Type:** Story · **Labels:** `Platform`, `Scope3`, `Site_Based`, `enhancement` · **Assignee:** Sagar Thacker
- **Roadmap:** "Updated Traci in Insights – connected to scopes 1, 2 and 3 data": **Should Have**, not started, Notion page blank. Realistically a Sprint 25 / Q1 candidate.
- **Blocked by:** ML-791 (Snowflake MCP), ML-792 (SQL Analyst on the harness), H2 and H3. Also the Data Ops Snowflake MCP auth ticket that Maxi asked Juan for, which is outside ML.

## Problem

Traci in Insights doesn't answer across a customer's Scope 1, 2 and 3 data together.

## Goal

A user in Insights asks Traci a question and gets an answer drawn from their Scope 1, 2 and 3 data, through a harness agent.

## Acceptance criteria

* The agent queries Scope 1, 2 and 3 data through the Snowflake MCP. Access is scoped to the user's tenant.
* It answers a fixed question set covering Scope 1–2 only, Scope 3 only and combined questions. Results are checked against known values.
* Charts render in chat (ML-790, visualization level 1).
* Runs are traced in Langfuse.

## Still open

* Is this the ML-792 SQL Analyst agent with more data sources, or a separate agent? The meeting leaned toward one agent with different MCP servers.
* Which Snowflake schemas hold Scope 1–2 vs Scope 3 data, and does the MCP expose them as resources?

---

## S. staghorn-coral: follow-ups from the ML-744 evaluation

**ML-744's conclusion** ([Notion write-up](https://app.notion.com/p/4161f4852006801f9e0eccf4b9e3705d), Victor, 2026-11-13):
- **Keep branching.** Across Cat 3 and Cat 6, the correct EF vendor was among the explored paths in 100% of branched rows. Branching forks where the input really is ambiguous, mostly the *Database* stage.
- **Use hybrid ranking (C).** It matches deterministic ranking (A) on accuracy, calls the judge on only about 10% of rows, and costs about the same as A. The judge-on-every-row method (B) is 10–16% more expensive and 1.5–3 s slower per row, and no more accurate.
- **Fix the score before adding categories.** All 11 Cat 3 misses are the same scoring artifact: the geometric mean lifts a path that has an extra near-certain rule-matched stage. This is already ticketed as **ML-805** (Victor, In Progress, Sprint 22).
- Guard limits never triggered.

**Gaps in the write-up worth raising with Victor:**
- **Baseline top-1 accuracy isn't reported**, so "branching beats baseline" isn't shown. Only top-k for branching is.
- **Cat 3 top-1 is 45%** for A and C. That's the ML-805 artifact; the re-run should show how far it recovers.
- **Finding 3 contradicts itself** on Cat 6: it says "within 5 points of A" and also "90% vs 90%, C ties A, B trails at 80%".
- **Small set:** 20 rows × 2 categories.
- **Baseline cost is missing** from Langfuse (empty `costDetails`), so baseline $/row is only an estimate.

### S1. ✅ **CREATED ML-811** — [staghorn-coral] Re-run the branching evaluation after the ranking fix
- **Type:** Story · **Labels:** `Scope3`, `experiment` · **Assignee:** Victor Zhang
- **Follows:** ML-744 · **Blocked by:** ML-805

**Goal:** Confirm that the ML-805 scoring fix closes the Cat 3 top-1 gap, and answer the question ML-744 left open: does branching beat baseline on top-1?

**Scope:**
* Re-run Cat 3 and Cat 6 on the same 20-row sets: baseline, A and C. Keep B only as a reference if it's cheap.
* Report **top-1 for baseline** next to top-1 and top-k for each branching method.
* Fix Langfuse pricing for baseline generations so that baseline $/row is measured, not estimated.
* Walk the ranking methods from one shared branching pass instead of three separate end-to-end runs. ML-744 says the separate runs cost about 3×.

**Acceptance criteria:**
* A comparison table of baseline vs A vs C for Cat 3 and Cat 6, with top-1, top-k, branch rate, judge-call rate, and $, seconds and calls per row.
* Cat 3's `3000709`-over-`3000414` pattern is gone, or explained.
* A yes/no recommendation on making C the default (S2), added to the ML-744 Notion page.

### S2. ✅ **CREATED ML-816** — [staghorn-coral] Make hybrid ranking (C) the default
- **Type:** Story · **Labels:** `Scope3`, `enhancement` · **Assignee:** Victor Zhang
- **Follows:** ML-739 (option C) and ML-742 (runner modes) · **Blocked by:** S1 (ML-811) confirming the recommendation

**Goal:** The deterministic flow ranks branched paths with hybrid (C) by default, which gets A's accuracy and only calls the judge on close rows.

**Scope:**
* Set C as the default ranking method in the runner config. Baseline and A/B stay available as runner modes for evals.
* Make the "too close to call" threshold for escalating to the judge configurable, and document it.
* Decide whether to keep B (judge on every row) as an eval-only mode or remove it.

**Acceptance criteria:**
* With no ranking method configured, the flow uses C.
* Judge calls are traced in Langfuse with the reason for escalating.
* The judge-call rate in wip is in line with S1 (about 10%).

**Out of scope:** adding more categories. This waits on S1, per ML-744's "fix the score first". The remaining-categories question is still open in the Sprint 23 draft, section B.

---

## Not drafted

| Gap | Why not |
|---|---|
| **Scope 3 Claude Skills** (Must, Maxi) | Owned by Maxi, with resources BE and Product only, not ML. Draft one if ML is expected to build the skills. |
| **Traci Ingestion Assistant V2** (Should, L, Sagar) | Has epic ML-631. The Notion page is blank, so the scope needs defining first. More likely Q1. |
| **LCA Enersys POC** (Must) | Covered by ML-807, ML-808 and A1a/A1b (Rohit). Plan in `projects/lca/q4-plan.md`. |
| **Fiscal-year implementation** | Waits on the ML-786 spike outcome (Sprint 23 draft, section B). |
| **Snowflake MCP auth, Traci secret keys** | Data Ops (Juan) and Maxi, outside the ML project. |
