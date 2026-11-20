# EnerSys LCA Phase 1: document ingest process (reference)

**Source:** Notion [📄 Document Ingest Process](https://app.notion.com/p/3ce1f485200680e79445f391b91ac4b1), last edited 2026-11-18. There's also a Figma overview board, "EnerSys-LCA-Bielsko". This file is a summary for ML context; Notion is the source of truth. Source documents are in the EnerSys SharePoint under `EnerSys LCA Phase 1/` (`Data Intake/`, `Working LCA Docs/`, `Audit/`).

**What it is:** the **manual** method the team used for EnerSys Phase 1. It produced a cradle-to-grave carbon footprint for one product: the **48V 4PzS500** motive lead-acid battery (Hawker Perfect Plus / PzS family), made at the **Bielsko, Poland** site. Each step lists its required outputs, method, formula or audit evidence, and how far AI could do it. **This is the process the LCA POC automates.** Methodology: **JRC** rules for industrial-battery carbon footprints, the Circular Footprint Formula (CFF), and Ecoinvent EFs. Impact category: climate change (kg CO₂e).

**Status for ML (Brian, 2026-11-20):** Sprint 23 LCA work stays as-is (ML-807, ML-808). This document is input for a **next-sprint (Sprint 24) research project**.

---

## Pipeline at a glance

| # | Stage | Main inputs | Hard part (manual today) | AI potential (per the doc) |
|---|---|---|---|---|
| 1 | Functional unit and parameters | Tech spec PDF, IEC 60254 test statement, life-cycle test docx | — | High: plain extraction |
| 2 | **Materials (BOM)** | Original BOM xlsx, then BOM with photos/docs, then structured "Materials emissions.xlsx" | Weights in pieces (PC), sub-assembly double counting, composition, process assignment, EF selection | Moderate to high. Process assignment and EF choice need judgment |
| 3 | Energy | Site water and gas FY2025, production FY2025 (kW) | Allocation of plant-level data | High: formulas |
| 4 | Waste | % scrap per BOM line (from the client) | Scrap rates aren't in the BOM | Maybe, moderate |
| 5 | Transport | Client supplier file (purchasing: vendor, qty, distance) | Built by hand by the client; distances from Google Maps or SAP | High if replicated per product family |
| 6 | Packaging | Photos of each packaging component on a scale | No packaging documentation exists | Moderate to high if replicated per product family |
| 7 | End of life (JRC) | BOM summary (mass; lead, PE and PP fractions; rest) | — (template) | High with the new EoL template |
| 8 | Discrepancy check | BOM vs finished-battery mass vs transport-tracked mass | Reconciliation judgment | High: formulas, plus a conservative rule |
| 9 | Final calculation and DQR | All stage totals | — | High |

## 1. Functional unit and parameters
From the **technical spec**, by extraction: capacity (Ah), energy capacity (Wh/kWh) and weight. Voltage is calculated (the doc writes `energy capacity * 2V`, which looks like a typo; probably cells × 2 V).
From the **test procedure** (IEC 60254 statement plus the accelerated life-cycle test): total life cycles (**1,500**) and EOL capacity (**80%**).

**Functional unit:** `Etotal (kWh) = Einitial × Ncycles × AVERAGE(SOHinitial, SOHEOL)`, with SOHinitial = 1. The table gives SOHEOL a "standard value" of **0.7**, while the test docs say **80%**. ⚠️ This inconsistency needs confirming.

## 2. Materials (BOM): the core of the work
**Per line item (Step 1):** object name, weight, material composition, process, % lost, waste, and weight in FU (kg or L, since the result is kg CO₂e per battery).
- **Weight:** sometimes stated, but mostly given in pieces (PC). For PC or missing weights, verify with photos of the item on a scale or with documentation. **Sub-assemblies are often listed alongside their own components**, which risks double counting. EnerSys confirmed which lines are independent.
- **Weight in FU:** `weight (kg)`, or `weight per PC × PCs` converted to kg or L.
- **Material composition:** stated, or confirmed by the client from documents or verbally. Otherwise use an industry-practice assumption (lower confidence). Mixed materials need a % split, so `weight × % × EF` per material.
- **Process:** which manufacturing step a material goes through. It's not in the BOM and has never been documented as a step; it was an unwritten judgment call. It isn't a number in the formula, but **it drives EF selection**. It's flagged as an automation candidate, and as a place where AI is likely to guess confidently.
- **% lost and waste:** not in the original BOM. The client supplied % waste per line, entirely by hand (Bielsko only tracks waste at site level). `waste = weight × % lost`. The doc is unsure AI can estimate % lost.
- **Energy of Charge:** a kWh line inside the BOM weights (charging energy, since it's a backup battery). It's subtracted from all totals and may be specific to this BOM.

**Summary outputs (Step 2), in kg and %:**
- `Total weight (without liquid) = SUM(weights) − liquids`. Liquids are sulphuric acid and distilled water, and the energy-of-charge line is removed too. This total is 100%.
- Cumulated **PP, PE and Lead** as separate items. JRC EoL needs product-specific values for these.
- `Rest = Total (without liquid) − PE − PP − Lead`
- `Total weight = SUM(Total FUs) − SUM(waste kg) − kWh for charge`

**LCA outputs (Step 3), per line:**
- **EF name:** from **Ecoinvent** (the doc says "v3.1"; confirm the version). It depends on composition, source (primary vs recycled), purity or alloy, production route, region and **process**. Matched by hand today. ➜ The doc suggests **reusing the Scope 3 AI EF matching tool** (`ef_match` / staghorn-coral).
- `Impact = weight in FU × EF value`
- **TeR/GeR rating** per line (technological and geographical representativeness, scored 1–5; JRC). Needed for the audit. Traces whether a data point is primary, secondary or assumed.

## 3. Energy
Plant-level data, allocated to the product by production share (24 kW rating). Building heat is excluded.
- `% allocation = product energy capacity / total production FY2025 (kW)`
- `allocated gas = % × total natural gas FY2025 (m³)`; `allocated electricity = % × total electricity FY2025 (MWh) × 1000`
- EFs are then chosen as in materials Step 3, with no DQR.

## 4. Waste
Quantities come from the BOM step. Each component needs a scrap/reject rate from the client; the BOM quantity is grossed up by that rate, and the difference is the waste. Most lines have none. `waste per line = % waste × weight in FU`, then assign an EF.

## 5. Transport (supplier → EnerSys site)
Inputs: transported mass, distance (Google Maps or SAP), mode, vehicle category and dataset. The client's purchasing department built a supplier file by hand, listing BOM materials from third parties, with vendors, quantities and distances.
- `weighted avg distance = Σ(distance × qty) / Σ(qty)` per material
- `tkm = weighted avg distance × actual weight (kg) / 1000`
- Then EF × tkm, with no DQR.

## 6. Packaging
No packaging documentation exists. The client photographed each primary, secondary and tertiary packaging component on a scale and confirmed the material. Outputs: component, material, weight in kg. Then EF plus TeR/GeR.

## 7. End of life (JRC)
Two stages: battery dismantling, then lead smelting and refining. These use the JRC default lead-acid recycling datasets (Tables 7 and 8) and **CFF** parameters (Table 3). The lead recycling credit sums five CFF terms (recycling or disposal, for "properly" and "non-properly returned" batteries, plus energy recovery = 0) to **−1.476 kg CO₂e per kg of lead**.
**Only four battery-specific inputs** are needed, from the BOM summary: total mass without liquid, and the lead, PE and PP fractions, with the rest going to landfill (a conservative assumption). The JRC default lead fraction of 0.82 is used only if the actual fraction is unknown. The incineration fraction defaults to 0. There's a reusable **Lead-Acid Battery EoL template** (xlsx). The EoL result is a **net credit**, i.e. negative.

## 8. Discrepancy check (mass balance)
Reconcile the BOM with the declared finished-battery mass, plus losses and waste. **A conservative rule:** missing mass gets the **highest EF** among the tracked materials.
- `missing = SUM(Total FUs) − (6 distilled-water lines + energy of charging) − SUM(transport-tracked actual weight)`
- The emissions for the missing mass are added to the transport total. (The doc says "LINK FILE HERE WHEN REVIEWED W PANOS", so this section isn't final.)

## 9. Final calculation and DQR
- `Total (kg CO₂e/battery) = Materials + Energy + Waste + Transport + Packaging + EoL`
- Normalized **per kW** (rated power) and **per kWh of functional unit** (`÷ Etotal`). The per-kWh figure is the headline result.
- `DQR = (TeR + GeR) / 2` per dataset. The battery DQR is weighted by each process's share of the footprint. The doc gives the rating range as 1–4 here and as 1–5 for TeR/GeR. ⚠️ Confirm.

---

## Takeaways for ML (input to Sprint 24 research)
- **It already answers most of ML-807's data-point questions** for one product: the boundary is effectively cradle-to-grave with JRC EoL, and the data inventory and sources are listed. ML-807/ML-808 can use it as the reference case.
- **Where automation pays off:** BOM restructuring (PC → kg, sub-assembly dedup, composition), **process assignment → EF selection** (reuse `ef_match`), and repeatable calculations (FU, energy allocation, tkm, EoL template, mass balance, DQR).
- **Where humans or the client stay in the loop:** weights that need photos, % scrap per line, packaging photos, supplier and distance data, and composition when it's undocumented. The tool should **ask for these and record the evidence** for the audit.
- **Every number needs an audit trail.** TeR/GeR and primary/secondary/assumption tagging are required per line, so the data model must carry source and evidence for each line.
- **Product-family replication:** several stages are marked "high confidence if we replicate per product family". Templates per family (EoL, packaging, transport) are a big lever.
- **Inconsistencies to resolve with the LCA team:** SOHEOL 0.7 vs 80%; the voltage formula; the Ecoinvent version; the DQR range (1–4 vs 1–5); and waste computed as `% × Total FUs` in one place vs `% × weight in FU` in another.
