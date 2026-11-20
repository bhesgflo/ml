# Q4 2026 roadmap: ML items

**Source:** Notion [🛣️ Q4 2026 Roadmap](https://app.notion.com/p/5d2fd754444940a8913d9ab9ab76f96c) (R&D / Product). These are the rows where Module or Resources includes `ML`.
**Snapshot:** 2026-11-20. Rows were last edited in Notion on 2026-11-19. Notion is the source of truth, so re-pull before planning.
**Note:** the shared link opens the "Default view", which shows only Must Have. This file also includes the two Should Have rows. No ML rows are Could Have or Won't Have.

Q4 ends Dec 31. After Sprint 23 (Nov 23 – Dec 7), about **2 sprints are left** in the quarter.

## Summary

| # | Need | Module | Priority | Size | Notion status | Owner | Jira |
|---|---|---|---|---|---|---|---|
| 1 | Traci Harness | ML | Must | — | In progress | Sagar | ML-790, ML-791, ML-792, ML-803 (see [harness](../projects/harness/)) |
| 2 | AI chat to refine answers | Compliance | Must | L | Not started | Ram | **none** |
| 3 | Fiscal-year aware AI documents | Compliance | Must | S | In progress | Ram | ML-786 (spike) |
| 4 | AI responses in first person, not "the undertaking" | Compliance | Must | XS | In progress | Ram | ML-787 |
| 5 | Automatic EF matching for deterministic flow | Scope 3 | Must | M | In progress | Victor | epic ML-712 (staghorn-coral) |
| 6 | Scope 3 Claude Skills | Scope 3 | Must | S | In progress | Maxi | **none** in ML (Resources: BE, Product) |
| 7 | LCA Enersys POC | Insights/Data Layer | Must | — | Not started | Rohit | epic ML-639; ML-807, ML-813–815, ML-818, ML-824 |
| 8 | Kion onboarding | Scopes 1 & 2 | Must | S | In progress | Chethan Patharla | epic ML-585 (unassigned) |
| 9 | Migrate Barnes Industrial to new Abbyy process | Scopes 1 & 2 | Must | M | In progress | Chethan Patharla | **none** (ML-206 is an old Barnes bug) |
| 10 | Updated Traci in Insights, connected to Scope 1, 2 & 3 data | Insights/Data Layer | Should | — | Not started | Sagar | **none** |
| 11 | Traci Ingestion Assistant V2 buildout | Scope 3 | Should | L | Not started | Sagar | epic ML-631 |

## In progress (6 Must Have + the harness)

**1. Traci Harness** (Sagar). The Notion page is blank: no goal, value or size. Everything known is in [`projects/harness/`](../projects/harness/). Rows 2, 10 and 11 depend on it.

**3. Fiscal-year aware AI documents** (Ram, S)
- Each AI document gets a fiscal-year tag when it's trained in. The tag is passed in at question time, so answers use only that year's documents.
- The same tag applies in the AI chat (row 2). The 2 Nov roadmap call said this is easy.
- It doesn't need the harness, so it fills ML's Compliance capacity until the harness is ready.
- Open: how users set or see the tag in the platform isn't defined yet. No dates are committed.

**4. First-person AI responses** (Ram, XS)
- Quick win, decided on the 2 Nov call: write in the company's own voice, not "the undertaking". Doesn't need the harness.
- Deeper "disclosure-ready" answers come next. They need internal review plus a design partner to define what that means (Hirschmann Auto was suggested). This also covers improving the relevance of AI Suggested References.
- Customers who asked: Hirschmann Auto directly. It's also in the CSM pain-point tracker.

**5. Automatic EF matching for deterministic flow** (Victor, M)
- Goal: AI suggests one or more paths through the deterministic flows. Rolled over from Q3.

**6. Scope 3 Claude Skills** (Maxi, S)
- Scope: Cat 3 support; Cat 5 support, including CSRD waste metrics mapped to KPI scorecards (for Lear); and Site/BU/LE standardization.
- Detail is on the Notion page "20260928 Scope 3 Process Alignment". Reason given: CSM-first and scalability.

**8. Kion onboarding** (Chethan, S). Goal: finish onboarding Kion.

**9. Barnes Industrial → new Abbyy process** (Chethan, M). Goal: migrate all files to the Abbyy IWG tenant and route customer bills to it.

## Not started

**2. AI chat to refine answers** (Ram, L, Must). **This is the "Compliance chat interface this quarter" from the harness meeting.**
- Problem: AI Suggested Response gives one answer that can't be refined. CS and customers say this is the biggest blocker, so users go to Claude instead.
- **Q4 scope is Phase 1 only:** follow-ups on an answer (shorter, change tone, search again, leave a document out) over documents already trained into Tracera, plus the fiscal-year tag. Phase 2 (past-year and cross-framework answers, Data Links via an MCP server) and "Later" (bulk prepare-with-Traci, report writing) are struck through on the page.
- Builds on the harness. Timing is still to be confirmed with backend and frontend. The page's product notes are a draft pending Sagar's sign-off.
- **Blocking UX decision:** chat per measurement, or always on? Sagar needs this to build the agent. Poppy and Colby are designing the flow.
- Effort is mostly FE and ML, with little BE. Final sizing isn't done. Hirschmann Auto offered to help build it.

**7. LCA Enersys POC** (Rohit, Must). Value: a proof of concept for product-level LCA footprints. The page is otherwise blank. **Brian confirmed on 2026-11-20 that Rohit owns it, with a working POC due by the end of Q4.** The plan is in [`projects/lca/q4-plan.md`](../projects/lca/q4-plan.md), and the tickets are **ML-807** (data points) and the BOM track **ML-813 → ML-814** in Sprint 23, then **ML-815, ML-818, ML-824** in Sprint 24. ML-808 was closed as a duplicate of ML-813 on 2026-11-20.

**10. Updated Traci in Insights** (Sagar, Should). Value: ask Traci in Insights and get answers drawn from Scope 1, 2 and 3 data together. The page is blank. In harness terms this needs the Snowflake MCP (ML-791), and the Data Ops Snowflake MCP/auth work.

**11. Traci Ingestion Assistant V2 buildout** (Sagar, Should, L). Value: messy raw Scope 3 files are cleaned up intelligently while the original data is preserved. The page is blank. The harness demo already showed ingestion with no domain knowledge.

## Things to resolve

1. **Harness timing vs Q4.** Row 2 is a Must, size L and not started, and it depends on the harness. The harness isn't deployed yet: Sagar still has to fix queuing and background jobs, add in-chat graphs and the Snowflake MCP, then port API/auth and deploy. With about 2 sprints left after Sprint 23, row 2 Phase 1 is at risk unless the per-measurement vs always-on UX decision lands soon. Rows 10 and 11 (Should) realistically slip to Q1.
2. **Sagar is the owner on 3 rows** (harness, Insights Traci, Ingestion V2) plus the Traci bug backlog (Sprint 23 draft, item C4). Two of these are size L.
3. ~~LCA ownership mismatch~~ Resolved 2026-11-20: Rohit owns LCA.
4. **Chethan Patharla** owns Kion and Barnes/Abbyy, but isn't in the team table in `TICKET_CONVENTIONS.md`. Jira also has a set of Abbyy process-skill tickets (ML-337…ML-355) in QA assigned to Chethan. Is Chethan on the ML team? Epic ML-585 (Kion) is unassigned.
5. **No ML ticket yet** for rows 2, 6, 9 and 10. Row 6 lists BE/Product as resources, so it may not need one.
6. **Blank Notion pages:** rows 1, 7, 10 and 11 have no goal or scope. The harness row could link to `projects/harness/`.
7. **Next tickets:** ML-786 is a spike, so the fiscal-year implementation ticket is the follow-up already listed in the Sprint 23 draft, section B.
