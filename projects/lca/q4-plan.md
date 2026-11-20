# LCA: Q4 2026 plan to an Enersys POC

**Owner:** Rohit Kumar · **Status:** Sprint 23 and 24 tickets exist (BOM track consolidated 2026-11-20). The rest of the plan is a draft.
**Roadmap:** Notion Q4 row "LCA Enersys POC", Must Have (see `plans/q4-2026-ml-roadmap.md`, row 7)
**Related:** epic ML-639 (LCA Agent), ML-640 (deepagents skeleton, in REVIEW), **ML-807** (data-point spike). BOM track: **ML-813** (hands-on spike) → **ML-814** (v0 template) → **ML-815** (skill and scripts) → **ML-824** (stage 1 in the harness agent). **ML-818** (LCA agent on the harness). ML-808 was closed as a duplicate of ML-813.

## Goal

**A working POC by Dec 31, 2026:** take a real Enersys product's BOM and produce a product carbon footprint, with every number traceable to a BOM line, an emission factor and a data source. Data gaps are flagged, not hidden.

✏️ **Proposed POC exit criteria.** Confirm these after ML-807.
* **Input:** real BOMs for 1–2 Enersys products, in the formats Enersys actually has.
* **Processing:** BOM → structured multi-level product tree, including packaging. Each part gets a material and mass, and is marked supplied or estimated. Each material gets an emission factor.
* **Output:** kgCO₂e per functional unit for the agreed boundary (cradle-to-gate is likely), broken down by component and life-cycle stage, with a list of data gaps.
* **Review:** CS walks through the result. Ideally Enersys reviews it as well.
* **Out of scope for the POC:** UI polish, other customers, full cradle-to-grave unless ML-807 says it's needed, and production hardening.

## Timeline

Q4 has **two working sprints left**. Sprint 25 covers the holidays, so plan to finish the POC by **Dec 18** and treat Sprint 25 as buffer.

| Sprint | Dates | Work | Exit |
|---|---|---|---|
| **23** | Nov 23 – Dec 7 | **ML-807** data-point review (3 d) in parallel with the BOM track: **ML-813** hands-on BOM spike (4 d), then **ML-814** v0 BOM template. Get sample Enersys BOMs and product data through CS **in the first week**. | **Checkpoint, ~Dec 4:** POC scope agreed (products, boundary, schema, approach). POC tickets drafted. |
| **24** | Dec 7 – 21 | Build the POC: **ML-815** BOM-cleaning skill, **ML-818** LCA agent on the harness, **ML-824** stage 1 in the agent, plus material → EF matching, the footprint calculation, and the gap and traceability report. | **Internal demo, ~Dec 18.** |
| **25** | Dec 21 – Jan 4 | Buffer: fixes from the demo, then a CS/Enersys walkthrough. | **POC done by Dec 31.** |

## Decisions and dependencies

1. **Sample data is the critical path.** Without real Enersys BOMs in week 1 of Sprint 23, ML-813 can only test against made-up files. Ask CS now. *(Owner: Brian / CS)*
2. **Ownership (decided 2026-11-20):** Rohit owns LCA, including ML-818 and ML-824. ML-824 gets updated with ML-813's recommendation before it starts. Victor stays on staghorn-coral (ML-805, S1, S2).
3. **Harness or not.** ML-818 builds LCA as a harness agent, but the harness isn't in wip yet. It needs H1–H3 in `drafts/sprint-23-gaps.md`. **Don't let the POC wait on it.** If the harness isn't in wip by the start of Sprint 24, build the POC as a standalone pipeline or notebook that reuses Axolotl services such as `ef_match`, and port it to the harness afterwards.
4. **EF matching.** Material-level matching may be able to reuse `ef_match` or the staghorn-coral deterministic flow. ML-807 should confirm this with Victor so we don't build a second matcher.
5. **ML-640.** It's been in REVIEW since Sprint 20. Close it, or merge only its non-LCA fixes (Sprint 23 draft, item C1).
6. **Rohit's load.** After the 2026-11-20 consolidation his Sprint 23 LCA work is ML-807, ML-813 and ML-814. ML-815, ML-818 and ML-824 are in Sprint 24. He also has ML-801, ML-802 and the Axolotl docs (ML-796–799) in Sprint 23.

## Reference: the manual EnerSys Phase 1 method

[`reference/enersys-document-ingest-process.md`](reference/enersys-document-ingest-process.md) summarizes the Notion page "Document Ingest Process". It covers the stage-by-stage manual LCA of the 48V 4PzS500 battery at Bielsko: functional unit, BOM materials, energy, waste, transport, packaging, JRC end of life, mass balance and DQR. **This is the process the POC automates.** Brian, 2026-11-20: it feeds a **Sprint 24 research project**. It's also referenced from ML-813 and ML-814 as the starting point for the BOM workflow and template.

## Rough Sprint 24 tickets (draft properly after the checkpoint)

* ~~[LCA] Ingest Enersys BOMs into the product-tree schema~~ → now ML-815 and ML-824
* [LCA] Match materials and parts to emission factors
* [LCA] Calculate the footprint per functional unit, with a breakdown by component and stage
* [LCA] Data-gap and traceability report for the POC output
* [LCA][SPIKE] Research: map the manual Phase 1 method to automation. For each stage: what to extract, calculate or ask the client for, and the audit evidence (TeR/GeR, primary/secondary) per line. Includes the inconsistencies listed in the reference.
