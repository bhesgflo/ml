# Sprint 23 — Draft tickets

**Sprint:** Sprint 23 (id 5841), Nov 23 – Dec 7, 2026
**Status:** A2–A6 created in Sprint 23 on 2026-11-17 (ML-800 – ML-804). A1 and A7 are still drafts.
**Source:** follow-ups found in Sprint 21/22 tickets: "out of scope" and "follow-up" notes, spike outcomes, and blocked work.

Legend: ✅ approve as-is · ✏️ edit · ❌ drop

---

## A. Ready to draft now

### A1. [LCA] Stage 1: BOM column mapping, validation and vision extraction
- **Type:** Story · **Labels:** `Supplier_Portal`, `enhancement` (copied from ML-640) · **Assignee:** Victor Zhang
- **Follows:** ML-640 (skeleton, REVIEW since Sprint 20), epic ML-639
- ⚠️ **Decide first:** ML-640 builds LCA in mountain-gorilla on deepagents. Sagar's new harness is now the target for agents (ML-792 moves SQL Analyst onto it). Should Stage 1 go on the ML-640 skeleton or on the harness?

**Goal:** Add the first real domain stage to the LCA agent: map a customer BOM's columns, validate them, and extract BOM data from images or PDFs.

**Context:** ML-640 built the LCA agent skeleton on deepagents in mountain-gorilla (`app/agents/lca/`, branch `st/feature/ml-640` off `feature/lca-intelligence`). It covers thread create/resume/streaming, a HITL round-trip and the S3 thread file backend. That ticket left the real stage-1 logic for "the next milestone".

**Scope:**
* BOM column mapping: map uploaded BOM columns to the LCA schema. Ask the user through HITL when a mapping is uncertain.
* Validation: check required columns, units and types. Report problems to the user rather than failing silently.
* Vision extraction: pull BOM tables out of image or PDF uploads into the same schema.
* Run each step through the existing HITL and S3 thread backend from ML-640.

**Acceptance criteria:**
* A reference BOM (CSV/XLSX) goes through mapping and validation end to end in wip.
* A reference BOM image or PDF is extracted into the same schema.
* An uncertain column mapping triggers a HITL approval.
* Runs are traced in Langfuse.

**Out of scope:**
* Stages 2–8.
* Frontend (forestelephant) Tool-UI cards.

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

## B. Waiting on results (draft once these land)

| Waiting on | Owner | Expected follow-ups |
|---|---|---|
| **ML-788** Complyrag RAG architecture spike | Ram | The ticket itself promises "follow-up implementation tickets": the tool design if we go granular, and latency fixes. |
| **ML-786** Fiscal-year filtering spike | Ram | A fix ticket for each gap it finds, if any. |
| **ML-744** staghorn-coral evaluation (write-up in Notion) | Victor | Adopt the winning ranking method as the default, remove the losers, and plan integration toward prod. ⚠️ I can't read Notion (connector not authenticated). Can you paste the recommendation? |
| **ML-713 / ML-714** MCP spike + harness tech design | Sagar | Next harness stages, e.g. the "walk" stage after ML-791's crawl. |
| **ML-789** harness background jobs | Sagar | Its "Still open" question: queueing a wake-up run behind an active run. |
| **ML-778** Remove Claude 4.x compatibility | Rohit | Already exists. The 5.5 soak runs "a few weeks" from ~Nov 10, so it probably fits Sprint 24. |
| staghorn-coral **remaining categories** | Victor | Covered so far: 4/9, 5, 6 (travel, hotel, spend), 7 activity, 11, 12 activity. **Not seen:** 1, 2, 3, 8, 10, 13, 14, 15, 7 spend and 12 spend. Are those planned, out of scope, or done elsewhere? |

## C. Backlog cleanup (no new tickets, just decisions)

1. **ML-640** (LCA skeleton) has been in **REVIEW since Sprint 20** and isn't in any sprint. It needs to be merged or reviewed before A1. Also: should it move from Sagar to Victor?
2. **ML-720** (complyrag empty responses) is assigned to Sagar but labelled Compliance, and has carried over 3 sprints. Its blocker ML-716 is Deployed. → Reassign to Ram?
3. **ML-794** (new Traci bug, Nov 12) is **unassigned**.
4. **Traci backlog:** about 15 open items (mostly prym and eos customer bugs, plus ML-632/634/595/597) are assigned to Sagar. With Sagar on the harness, who owns Traci now: Rohit (Scope 3) or Ram, who fixed the recent Traci bugs?
5. **ML-308** CDP spike has been **In Progress since March** with no sprint. Close it or hand it to Ram?
6. **Compliance backlog** (ML-345, 347, 348, 349 for summarization expansion to CDP, TCFD, IFRS, GRI and UNGC) is unassigned. These could go to Ram after onboarding (ML-785).
7. **ML-559** Bonobo Documentation has carried over **8 sprints** in To Do. It could be split per package like the Axolotl docs (ML-796–799).
