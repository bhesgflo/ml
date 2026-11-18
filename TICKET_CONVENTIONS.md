# ESGFLO ML — Ticket Conventions

Derived from Sprint 21 (Oct 26 – Nov 8, 2026) and Sprint 22 (Nov 9 – 23, 2026) on the ESGFLO ML board.

## Jira reference

| Thing | Value |
|---|---|
| Site | https://esgflo.atlassian.net (cloudId `32a726ae-828a-4453-9761-ea7862f2765d`) |
| Project | **ESGFLO ML**, key `ML`, id `10008`, board id `8` |
| Sprint field | `customfield_10020` |
| Story points field | `customfield_10016` (rarely used) |
| Sprints | 2 weeks, Monday start, named `Sprint N`. Sprint 22 = id 5804 (active), **Sprint 23 = id 5841** (Nov 23 – Dec 7) |

**Issue types:** Story `10007` (default, ~85% of tickets), Task `10017` (infra/ops chores), Bug `10019` (customer-reported issues), Epic `10000`.

**Workflow:** To Do → In Progress → REVIEW → QA → Deployed / Done / Closed

**Priority:** almost always `Medium`. Use High or Highest only for real P1 escalations.

### Team

| Name | Account ID | Primary focus (per Brian) | Secondary / observed |
|---|---|---|---|
| Sagar Thacker | `712020:3043867d-69ba-44f2-b6b6-e54dca340b7c` | **Harness** | Scope 3, Compliance (mt-gorilla, Traci, distance calc, Abbyy) |
| Victor Zhang | `712020:64377384-3b02-4f8b-96dd-4c0f595e34c8` | **LCA** | staghorn-coral (S3 Deterministic Flow) in S21/S22 |
| Ram Gandikota | `712020:6c4e19ac-306b-4617-b192-49f7d2b0b1db` | **Compliance** (Complyrag) | Traci bugs, Vaquita/pptxgen |
| Rohit Kumar | `712020:9832ba19-d061-4931-bebf-8fdaddd3c5fd` | **Scope 3** | Axolotl/Bonobo platform, EPD/Supplier, model upgrades, docs |
| Brian Hirst | `712020:fefe2da8-4fbc-4d33-8050-f91badcea641` | Manager. Takes infra/Terraform/IAM Tasks himself | |

**Assignment rule:** assign each ticket to the primary owner for its area. People do work outside their area, so for **follow-on / next-phase work, assign whoever did the prior ticket** (check the parent or predecessor ticket's assignee).

Other reporters: Amy Ryu, Chiara Meacci, Sandra Kristanto (usually customer bugs).

### Open epics (parent)

| Key | Epic |
|---|---|
| ML-712 | S3 - Deterministic Flow (staghorn-coral) |
| ML-631 | Traci Ingestion Assistant |
| ML-639 | LCA Agent |
| ML-585 | Kion Onboarding |
| ML-344 | Report Writing Agent |
| ML-343 | Agents |
| ML-342 | Abbyy 2.0 |
| ML-301 | Abbyy Post-Processing |
| ML-208 | Abbyy-Replacement |
| ML-158 | Supplier-DD-MVP |

Cross-project parents also appear, e.g. `FND-3120` and `S3-816`. Most non-staghorn tickets have **no** epic.

## Titles

Pattern: **`[component][sub-component] Imperative description`**, in sentence case with no trailing period.

- Component prefixes are repo or service codenames: `[staghorn-coral]`, `[Axolotl]`, `[Bonobo]`, `[Complyrag]`, `[harness]`, `[mt-gorilla]`, `[Traci]`, `[Vaquita]`, `[Abbyy]`, `[EPD]`. You can stack them, e.g. `[Axolotl][Complyrag]` or `[Axolotl][Climate-Risk]`.
- Add `[SPIKE]` for research or timeboxed investigations, and `[Tech-Design]` for design docs.
- Customer bugs follow `[P1|P2|P3] [customer] description`, e.g. `[P3] [eos] Traci asks a follow up question…`.
- Infra tasks often have no prefix and name the environments in parentheses, e.g. `… (wip, preview and live)`.
- Recurring work is split into one ticket per unit with a parallel title, e.g. `Axolotl Documentation: complyrag` and `[staghorn-coral] Add Category 7 (Activity) to the deterministic flow`.

## Labels

Use one or more **product area** labels plus one **work type** label.

- **Product area:** `Scope3`, `Platform`, `Compliance`, `Supplier_Portal`, `Site_Based`
- **Work type:** `enhancement`, `chore`, `research` (spikes), `experiment` (staghorn experiments), `design`
- **Customer** (bugs only): lowercase customer name, e.g. `eos`, `enersys`, `prym`

## Description templates

Bold headings (`**Goal:**`) and `##` headings are both used. Bullets are short. The writing is plain and specific: it names files, functions, repos and environments in `code`.

### Story / enhancement
```
**Goal:** One sentence on what changes and why.

**Context:** (optional) Current behavior / why now.

**Scope:**
* Concrete change 1
* Concrete change 2

**Acceptance criteria:**
* Observable, testable outcome

**Out of scope:** (optional)
* ...
```

### Spike (`[SPIKE]`, label `research`)
```
**Goal:** The decision to make.

**Context:** What's true today and why it's unclear.

**Questions to answer:**
* ...

**Options to compare:** (optional)
1. **Option A:** ...

**Outcome:** A write-up / recommendation + follow-up tickets.

**Timebox:** N days.
```

### Infra Task
```
**Context**
Why this is needed.

**Scope**
* Steps, environments (wip → preview → live)

**Acceptance criteria**
* ...
```

### Harness-style (Sagar's)
```
## Problem
## Goal
## Acceptance criteria
## Still open / Blocked by
```

### Customer bug (intake form)
```
Customer: <name>
Environment: <scope 3 | ...>
Timeline: P1|P2|P3
Type: bug
Description: ...
Desired Outcome: ...
Slack Message Link: <url>
```

### Engineering bug (found in code)
`## Problem` → `## Why / Root cause` → `## Impact` → `## Reproduction` → `## Proposed fix` → `## Origin`

## Ticket-creation workflow (agreed with Brian, 2026-11-17)

1. Draft each batch as markdown in `drafts/sprint-NN.md`: title, type, labels, assignee and description per ticket.
2. Brian reviews and edits the draft.
3. Only after approval, create the tickets in Jira in the target sprint. Then record the created keys back in the draft file.

Policies for now:
- **Story points:** leave them blank. Brian wants to add them gradually later.
- **Epics:** don't set a parent unless Brian asks. He'll start directing epics in a few sprints.
- **Assignee:** always set, per the assignment rule above.

## Notes

- Story points are mostly unset. Only some recent Complyrag tickets have them (1–3).
- Tickets often carry over across several sprints.
- Environment names: **wip** (dev), **preview** (staging), **live** (prod). Branches: `develop` / `release` / `main`.
