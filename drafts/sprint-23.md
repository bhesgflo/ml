# Sprint 23 — Draft tickets

**Sprint:** Sprint 23 (id 5841), Nov 23 – Dec 7, 2026
**Status:** A2–A6 created in Sprint 23 on 2026-11-17 (ML-800 – ML-804). A1a/A1b (LCA, rewritten for the harness), A7 are still drafts. A8/A9/A10 were created on 2026-11-20 as ML-807/ML-808/ML-812. More Sprint 23 drafts are in `drafts/sprint-23-gaps.md`.
**Source:** follow-ups found in Sprint 21/22 tickets: "out of scope" and "follow-up" notes, spike outcomes, and blocked work.

Legend: ✅ approve as-is · ✏️ edit · ❌ drop

---

## A. Ready to draft now

### A1a. [harness][LCA] Set up the LCA agent on the harness
- **Type:** Story · **Labels:** `Platform`, `Supplier_Portal`, `enhancement` · **Assignee:** Rohit Kumar (moved from Victor 2026-11-20; Sagar supports on the harness side)
- **Follows:** ML-640 (deepagents skeleton in mountain-gorilla, REVIEW since Sprint 20), epic ML-639. **Decision (2026-11-17):** LCA is built on the harness, not on the ML-640 skeleton.

## Problem

The LCA agent skeleton (ML-640) was built in mountain-gorilla on deepagents. Agents are now moving to the harness: ML-792 moves SQL Analyst there to get its threads, approvals, step history and tracing. Building LCA stages on the ML-640 skeleton would mean migrating it later.

## Goal

LCA is defined as a harness agent (`agents/lca/` with `agent.yaml` and prompts), with the same baseline ML-640 proved out. Stage 1 logic can then be added directly on the harness.

## Acceptance criteria

* The LCA agent is defined in `agents/lca/` and runs in wip through the harness.
* A thread can be created, resumed and streamed.
* A user can upload a file to a thread, and the agent can read it. This replaces ML-640's S3 thread backend.
* A smoke-test tool that needs approval round-trips through the harness approval rules (approve and reject).
* Runs are traced in Langfuse.
* Anything from ML-640 worth keeping is listed in this ticket, e.g. the `routes_run.py` stale-run fix and the `env.prod-us.json` inference-profile fix. Those may matter outside LCA.

## Still open

* How does an agent add custom (non-MCP) tools on the harness? Stage 1 needs them for column mapping, validation and vision extraction. Check with Sagar against the ML-714 tech design.
* What happens to ML-640 and the `feature/lca-intelligence` branch: close without merging, or merge only the non-LCA fixes?

---

### A1b. [LCA] Stage 1: BOM column mapping, validation and vision extraction
- **Type:** Story · **Labels:** `Supplier_Portal`, `enhancement` · **Assignee:** Rohit Kumar (moved from Victor 2026-11-20)
- **Blocked by:** A1a, and A9 (ML-808). ✏️ Rewrite this ticket with ML-808's chosen approach and schema before creating it. It's planned for Sprint 24.

## Problem

The LCA agent has no domain logic yet. Stage 1, getting a customer's BOM into a clean, validated LCA schema, is the first of the 8 stages and everything after it depends on it.

## Goal

The LCA agent on the harness takes a customer BOM (CSV, XLSX, image or PDF), maps its columns to the LCA schema, validates the result, and asks the user when it isn't sure.

## Acceptance criteria

* **Column mapping:** the uploaded BOM's columns are mapped to the LCA schema. An uncertain mapping goes to the user for approval through the harness approval rules.
* **Validation:** required columns, units and types are checked. Problems are reported to the user in the thread, never dropped silently.
* **Vision extraction:** BOM tables in images or PDFs are extracted into the same schema.
* A reference BOM in CSV/XLSX and one as an image or PDF each go through stage 1 end to end in wip.
* Runs are traced in Langfuse.

## Out of scope

* Stages 2–8.
* Frontend Tool-UI cards.

---

### A2. ✅ **CREATED ML-800** — [mt-gorilla] Point prod-us at the US-region distance calculation lambdas
- **Type:** Task · **Labels:** `Scope3` · **Assignee:** Brian Hirst
- **Follows:** ML-763 (deploy distance calc to US, live), ML-762 (MG IAM for renamed lambdas), ML-758 (parameter store names)

**Context**
ML-763 deploys the distance calculation stack to the US region on the live account. Mountain Gorilla already runs a US deployment (`env.prod-us.json`). Once the US lambdas exist, the US deployment has to call them rather than the EU ones, and it needs permission to.

**Scope**
* Add the US-region distance calculation lambda ARNs to the Mountain Gorilla prod-us ECS task role. Use the same narrow permission as ML-762, not a wildcard.
* Make sure the US-region parameter store entries from ML-758 exist and that prod-us reads them.
* Run a distance calculation from prod-us and confirm in CloudWatch that it hits the US lambdas.

**Acceptance criteria**
* prod-us invokes the US-region distance calculation lambdas successfully.
* The EU deployment is unaffected.
* The Terraform changes are merged.

---

### A3. ✅ **CREATED ML-801** — [axolotl] ef_match: fix the v-prefixed prompt version defaults
- **Type:** Bug · **Labels:** `Platform`, `Scope3` · **Assignee:** Rohit Kumar
- **Follows:** ML-721. It was closed as investigated, but Rohit's comment says the defaults are still in the code.

## Problem
Five `ef_match` prompt-version defaults are `"v1.0.0"`. `LocalPromptTemplate` adds its own `v`, so these resolve to `vv1.0.0`, which doesn't exist. Prod is unaffected today because the prod config sets every version explicitly. The bug will show up as soon as a config omits a version or a new pipeline is added without one, and the pipelines' quiet fallbacks would hide it.

## Proposed fix
Make `LocalPromptTemplate` accept both forms by stripping a leading `v`. This is option 2 in ML-721, and it also protects any existing configs that use the prefix. Then change the defaults to `"1.0.0"` for consistency.

## Acceptance criteria
* A test loads each pipeline's default prompt names and versions against the real `prompts/templates` folder and passes.
* `"1.0.0"` and `"v1.0.0"` both resolve to `templates/<name>/v1.0.0`.
* No default resolves to a path that doesn't exist.

---

### A4. ✅ **CREATED ML-802** — [Axolotl][SPIKE] Find the cause of empty Bedrock responses
- **Type:** Story · **Labels:** `Platform`, `research` · **Assignee:** Rohit Kumar
- **Follows:** ML-716 and ML-720 both list the cause as out of scope. ML-315 (Apr) checked the same symptom in EPD.

**Goal:** Find out why Bedrock sometimes returns `content: []` with a handful of output tokens, so we fix the cause rather than only retrying.

**Context:** ML-716 made `ef_match` and `epd` handle empty responses (`extract_text` / `require_text`), and ML-720 does the same for complyrag. Both treat the cause as unknown. The symptom has come up before (ML-315). Since the 5.5 migration, models now use effort/thinking and automatic tool choice, which may change how often this happens.

**Questions to answer:**
* How often does it happen per service and per model, before and after the 5.5 migration? Check the logs and Langfuse.
* Do empty responses correlate with prompt size, thinking/effort, tool choice, stop reason or output-token limits?
* Is it documented Bedrock/Anthropic behaviour, for example an empty `end_turn` after tool results?
* Can we reproduce it on demand?

**Outcome:** A short write-up with the likely cause(s), plus follow-up tickets if there's a fix beyond retrying.

**Timebox:** 2 days.

---

### A5. ✅ **CREATED ML-803** — [harness] Answer every pending tool call when a HIL approval is auto-cancelled
- **Type:** Story · **Labels:** `Platform`, `enhancement` · **Assignee:** Sagar Thacker
- **Follows:** ML-770, which fixed this in mt-gorilla's default engine. It noted that "the native engine has a related gap, out of scope because that engine is being reworked." Related backlog item: ML-617.

## Problem
When a thread is paused at a HIL approval and the user sends a normal message, the waiting tool calls are auto-cancelled. ML-770 found that auto-approved calls in the same turn were left with no `ToolMessage`, which leaves orphaned tool calls in the checkpoint. It fixed this for the default engine only.

## Goal
The harness, and any engine it still uses, answers every unanswered tool call in the paused message when a HIL approval is auto-cancelled.

## Acceptance criteria
* A turn with one gated call and one auto-approved call, followed by a normal user message, leaves no orphaned tool calls in the checkpoint.
* The model gets an accurate cancelled or skipped message, not "recursion limit reached".
* There is a regression test for that case.

---

### A6. ✅ **CREATED ML-804** — [pptxgen] Run the unit tests in CI on pull requests
- **Type:** Story · **Labels:** `Site_Based`, `chore` · **Assignee:** Ram Gandikota
- **Follows:** ML-745 (unit tests) and ML-747 (deploy workflows)

**Goal:** Run the pptxgen tests from ML-745 automatically, so a broken batch or validation change is caught before deploy.

**Scope:**
* Add a GitHub Actions workflow that runs the pptxgen tests on PRs touching `pptxgen/**`, with no AWS credentials.
* Use the same toolchain setup as the ML-747 deploy workflows: Python 3.12 and `uv`.
* Make a failing test block the merge, using the branch protection on `develop`.

**Acceptance criteria:**
* A PR touching `pptxgen/**` runs the tests and shows the result on the PR.
* A PR that doesn't touch `pptxgen/**` doesn't trigger it.
* The workflow is documented in `pptxgen/README.md`.

---

### A7. [pptxgen] Add removal workflows for temporary stages
- **Type:** Story · **Labels:** `Site_Based`, `chore` · **Assignee:** Ram Gandikota
- **Follows:** ML-747, which listed `pptxgen.remove.*` as an optional follow-up. Lower priority, so a good candidate to defer.

**Goal:** Allow a pptxgen stack to be removed from GitHub Actions, so temporary `dev-<name>` stages don't keep costing money.

**Scope:**
* Add a manual `pptxgen.remove.dev.yml` workflow that runs `sst remove --stage <stage>` in the wip account.
* Allow only `dev-*` stages. Never remove `dev`, `staging` or `prod`.
* Reuse the ML-747 auth and toolchain setup.

**Acceptance criteria:**
* A `dev-<name>` stage can be removed from the Actions tab, and its SQS, Lambda and Step Functions resources are gone.
* The workflow refuses `dev`, `staging` and `prod`.

---

### A8. ✅ **CREATED ML-807** — [LCA][SPIKE] Enersys LCA: review the key data points and what Enersys can supply
- **Type:** Story · **Labels:** `Supplier_Portal`, `research` · **Assignee:** Rohit Kumar
- **Context:** Q4 roadmap row "LCA Enersys POC" (Must Have, owner Rohit). Enersys is about to start building product LCAs. Plan: `projects/lca/q4-plan.md`. Related: epic ML-639, A1a/A1b, and ML-551 (an earlier Enersys ingestion bug).

**Goal:** Decide what data a product LCA for Enersys needs, which of it Enersys can supply, and what the Q4 POC will cover.

**Context:** We have an LCA agent design (ML-639, 8 stages) and a harness skeleton plan (A1a/A1b). We haven't yet checked them against a real customer's data. Enersys makes batteries, so materials such as lead, lithium, acid and plastic casings, recycled content and end-of-life will matter.

**Questions to answer:**
* Which **key data points** are needed, and is each required, optional or defaultable? At least: functional unit, system boundary, BOM (components, materials, mass), material composition and recycled content, supplier and origin, manufacturing energy per unit, transport, packaging, use phase, end of life, and emission factor / LCI source.
* **System boundary for the POC:** cradle-to-gate or cradle-to-grave? Which standard do we align to (ISO 14067 / GHG Protocol Product Standard)?
* Does Enersys need anything regulatory, e.g. battery carbon-footprint declarations under the EU Battery Regulation? Confirm with CS and Product.
* For each data point, **where does it come from at Enersys** (ERP, PLM, supplier declarations, spreadsheets), and how complete is it?
* Which emission factor sources do we have for these materials, and can the existing EF matching (`ef_match`, staghorn-coral) be reused?
* Which **one or two Enersys products** should the POC target?

**Outcome:**
* A data-point inventory table (data point · required? · Enersys source · availability · fallback), added to ML-639.
* A proposed POC scope: products, boundary and output.
* Follow-up POC tickets.
* Agreed with Brian by the Sprint 23 checkpoint (see the plan).

**Timebox:** 3 days.

---

### A9. ✅ **CREATED ML-808** — [LCA][SPIKE] BOM ingestion: turn unstructured BOMs into a structured product tree
- **Type:** Story · **Labels:** `Supplier_Portal`, `research` · **Assignee:** Rohit Kumar
- **Context:** The POC has to start from a customer BOM. Plan: `projects/lca/q4-plan.md`. This overlaps A1b (stage 1 BOM mapping, validation and vision extraction). A9 decides the approach and A1b builds it.

**Goal:** Choose how we turn a customer's BOM into a structured, multi-level product tree we can calculate a footprint on. The BOM covers everything that goes into a product, from components and sub-assemblies to packaging such as the box.

**Context:** BOM ETL is highly unstructured. BOMs come as ERP exports, Excel sheets with no fixed layout, PDF spec sheets and sometimes images. They often lack mass or material, mix units, and flatten or repeat the hierarchy. Packaging and consumables are often missing entirely.

**Questions to answer:**
* What do real Enersys BOMs look like? Formats, hierarchy depth, and which fields are present. ⚠️ Needs sample BOMs, via CS.
* What's the **target BOM schema**? Assembly → sub-assembly → part, with quantity, unit, mass, material, supplier, origin, and a packaging flag.
* How do we fill gaps, e.g. mass from spec sheets, or material inferred from part descriptions? How do we flag what's estimated vs supplied?
* How do parts and materials map to emission factors? Can `ef_match` / staghorn-coral do material matching?
* Where does a human review, and what does the user approve?

**Options to compare:**
1. **Deterministic parsers** plus per-customer mapping templates.
2. **LLM extraction**, file by file, into the schema, including vision for PDFs and images.
3. **A harness agent in the sandbox** that writes per-file transformation code, as in the data-ingestion demo, with approvals for uncertain mappings.
4. **Reuse the Traci Ingestion Assistant** pipeline.

**Outcome:**
* A recommended approach, the target schema, and 2–3 real Enersys BOMs run through a throwaway prototype, with accuracy and gaps noted.
* Follow-up POC tickets.

**Timebox:** 4 days.

---

### A10. ✅ **CREATED ML-812** — [harness][Bonobo][SPIKE] Get Insights output from the harness into Bonobo
- **Type:** Story · **Labels:** `Platform`, `research` · **Assignee:** Sagar Thacker, with Rohit Kumar consulted (he has done most of the Bonobo work)
- **Context:** Brian, 2026-11-20: start targeting incorporating Insights output from the harness into Bonobo. It will take several steps, and this spike plans them. Related: R3 (Traci in Insights over Scope 1–3 data, in `drafts/sprint-23-gaps.md`), ML-791, ML-792, ML-809, ML-810.
- ✏️ Sagar already has H1, ML-790, ML-791, ML-792, ML-803 and ML-810 this sprint. This may have to start late in the sprint or carry over.

**Goal:** Decide how the Insights output a harness agent produces (answers, charts, generated files and reports) gets into Bonobo, and break that into step-by-step tickets that follow the harness milestones.

**Context:** Bonobo runs our workflows (complyrag gap assessment, EPD, climate risk, AI summary, EF recommendations). It has request contracts (ML-643), workflow configs (ML-690), a notification service (ML-691) and Langfuse tracing. The harness is the new agent layer. Today its outputs live only in its own session store: files in S3 per session, versioned, plus threads in Postgres. The harness meeting agreed that bulk runs stay as workflows, so Bonobo stays, and the two need a defined hand-off.

**Questions to answer:**
* **What output, specifically?** Chat answers, charts (PNG now; HTML and dashboards later per the 4 visualization levels), generated files (docx, xlsx, HTML/PDF reports), and the queries and sources behind them. Which of these does Insights need Bonobo to hold or act on?
* **What does Bonobo do with it?** Store it as the system of record, serve it to the Insights UI, trigger a workflow (e.g. scheduled regeneration of a report, notifications), or a combination?
* **Contract:** what does a harness → Bonobo request look like, following Bonobo's existing request-contract and config patterns? Who owns the schema?
* **Sync or async:** does the agent call Bonobo directly in a run, or hand off through a background job or event? Is the existing event bus (ML-761) an option?
* **Storage and tenancy:** copy from harness S3 into Bonobo, or reference it? How do tenant isolation, retention and versioning carry over?
* **Tracing:** how is a harness trace linked to the Bonobo workflow trace in Langfuse (tenant, service, user)?

**Options to compare:**
1. **Bonobo API:** the harness pushes finished outputs to a new Bonobo endpoint, and Bonobo is the system of record.
2. **Bonobo as an MCP server/tool:** the agent calls tools such as "save insight" or "run workflow" when it decides to, behind approvals.
3. **Events:** the harness emits an "insight produced" event and Bonobo consumes it.
4. **Shared storage:** Bonobo indexes the harness S3 outputs in place.

**Outcome:**
* A recommended option, with a sketch of the contract and a data-flow diagram.
* A sequence of follow-up tickets, each tied to the harness milestone it needs: local, wip (ML-809), auth (ML-810), Snowflake MCP (ML-791).
* Agreement with Rohit on the Bonobo side.

**Timebox:** 3 days.

---

## B. Waiting on results (draft once these land)

| Waiting on | Owner | Expected follow-ups |
|---|---|---|
| **ML-788** Complyrag RAG architecture spike | Ram | The ticket itself promises "follow-up implementation tickets": the tool design if we go granular, and latency fixes. |
| **ML-786** Fiscal-year filtering spike | Ram | A fix ticket for each gap it finds, if any. |
| **ML-744** staghorn-coral evaluation (write-up in Notion) | Victor | ✅ Read 2026-11-20: keep branching, use hybrid (C), fix the score first. The score fix is ML-805 (in progress). Follow-ups drafted as **S1** (re-run the eval) and **S2** (make C the default) in `drafts/sprint-23-gaps.md`. |
| **ML-713 / ML-714** MCP spike + harness tech design | Sagar | Next harness stages, e.g. the "walk" stage after ML-791's crawl. |
| **ML-789** harness background jobs | Sagar | Its "Still open" question: queueing a wake-up run behind an active run. |
| **ML-778** Remove Claude 4.x compatibility | Rohit | Already exists. The 5.5 soak runs "a few weeks" from ~Nov 10, so it probably fits Sprint 24. |
| staghorn-coral **remaining categories** | Victor | Covered so far: 4/9, 5, 6 (travel, hotel, spend), 7 activity, 11, 12 activity. **Not seen:** 1, 2, 3, 8, 10, 13, 14, 15, 7 spend and 12 spend. Are those planned, out of scope, or done elsewhere? |

## C. Backlog cleanup (no new tickets, just decisions)

1. **ML-640** (LCA skeleton) has been in **REVIEW since Sprint 20** and isn't in any sprint. Since LCA is moving to the harness (A1a), decide whether to close it or merge only its non-LCA fixes.
2. **ML-720** (complyrag empty responses) is assigned to Sagar but labelled Compliance, and has carried over 3 sprints. Its blocker ML-716 is Deployed. → Reassign to Ram?
3. **ML-794** (new Traci bug, Nov 12) is **unassigned**.
4. **Traci backlog:** about 15 open items (mostly prym and eos customer bugs, plus ML-632/634/595/597) are assigned to Sagar. With Sagar on the harness, who owns Traci now: Rohit (Scope 3) or Ram, who fixed the recent Traci bugs?
5. **ML-308** CDP spike has been **In Progress since March** with no sprint. Close it or hand it to Ram?
6. **Compliance backlog** (ML-345, 347, 348, 349 for summarization expansion to CDP, TCFD, IFRS, GRI and UNGC) is unassigned. These could go to Ram after onboarding (ML-785).
7. **ML-559** Bonobo Documentation has carried over **8 sprints** in To Do. It could be split per package like the Axolotl docs (ML-796–799).
