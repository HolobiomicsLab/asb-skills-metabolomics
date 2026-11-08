---
name: novel-metabolite-formula-annotation
description: Use when annotating molecular formulas de novo from tandem MS spectra
  without spectral/structure databases, or when claiming a molecular formula is novel
  (absent from PubChem/ChemSpider).
when_to_use_negative:
- The goal is compound structure identification rather than formula annotation — use
  CSI:FingerID structure search instead, since searching a novel-formula compound
  in a structure database yields incorrect hits that often go unnoticed.
- Compounds are predominantly below 500 Da and database hits are expected — SIRIUS
  already performs excellently there, so the novel-formula pipeline (strict filters,
  database-absence checks) adds little.
- A complete evaluation via full structural elucidation is required — the paper states
  this is experimentally infeasible for novel formulas.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SIRIUS
  role: Computes fragmentation trees and isotope-pattern-based molecular formula candidates
    from MS/MS spectra; ZODIAC is integrated within it
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Reranks SIRIUS's top 50 formula candidates network-wide via Gibbs sampling;
    its scores (>=0.999) and similarity network (>=20 compounds) define the strict
    novel-formula filters
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS 2.4.0 (modified)
  role: Preprocessing of mzML/mzXML input files and discarding low-quality MS/MS spectra
    in the ZODIAC workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: PubChem
  role: PUG REST fastformula endpoint queried to count compound identifiers per novel
    formula, verifying database absence
- name: ChemSpider
  role: Second public structure database for absence checks; requires account credentials,
    so deferred to expert review
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:b03bcdaae8fc3f038153693399d9f050c68ea18f66e0d2665ae74612fe357d87
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - ChemSpider
  - OpenMS 2.4.0 (modified)
  - PubChem
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# novel-metabolite-formula-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate molecular formulas de novo from LC-MS/MS data with SIRIUS and ZODIAC, apply strict quality filters to nominate novel formulas, and verify their absence from public structure databases such as PubChem and ChemSpider via REST queries.

## When to use

Use when annotating molecular formulas de novo from tandem MS spectra without spectral/structure databases, or when claiming a molecular formula is novel (absent from PubChem/ChemSpider).

## When NOT to use

- The goal is compound structure identification rather than formula annotation — use CSI:FingerID structure search instead, since searching a novel-formula compound in a structure database yields incorrect hits that often go unnoticed.
- Compounds are predominantly below 500 Da and database hits are expected — SIRIUS already performs excellently there, so the novel-formula pipeline (strict filters, database-absence checks) adds little.
- A complete evaluation via full structural elucidation is required — the paper states this is experimentally infeasible for novel formulas.

## Inputs

- mzML or mzXML LC-MS/MS files (e.g. tomato, diatoms, dendroides, NIST1950, mice stool datasets)
- SIRIUS molecular formula candidate lists (top 50 per compound)
- Deposited novel-formula list (compounds_with_novel_molecular_formulas.csv, Supplementary Tables 3 and 4; 62 formulas)

## Outputs

- ZODIAC-scored molecular formula annotations per compound
- Filtered list of novel molecular formulas per dataset (e.g. 3 tomato + 59 diatoms = 62 formulas)
- pubchem_novel_formula_verification.csv with one row per formula and a PubChem hit count (expected 0 for all 62 rows)

## How to apply

Process input mzML/mzXML files with the modified OpenMS 2.4.0 ZODIAC workflow, discarding low-quality MS/MS spectra. Run SIRIUS to compute fragmentation trees and candidate formulas, then run ZODIAC, which reranks the top 50 SIRIUS candidates per compound using Bayesian statistics with Gibbs sampling (burn-in 1,000 epochs, sampling 2,000 epochs, ten parallel chains), optionally using spectral-library hits as anchors. To nominate novel formulas, apply the strict filters: minimum ZODIAC score 0.999, at least 95% of MS/MS spectrum intensity explained by SIRIUS, and at least one molecular formula connected to 20 or more compounds in the ZODIAC similarity network. For each filtered formula (e.g. C24H47BrNO8P), query the PubChem PUG REST fastformula endpoint and record the returned compound identifier count, expecting 0 hits; treat the analogous ChemSpider check as deferred when its API credentials are unavailable.

## Related tools

- **SIRIUS** (Computes fragmentation trees and isotope-pattern-based molecular formula candidates from MS/MS spectra; ZODIAC is integrated within it) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Reranks SIRIUS's top 50 formula candidates network-wide via Gibbs sampling; its scores (>=0.999) and similarity network (>=20 compounds) define the strict novel-formula filters) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS 2.4.0 (modified)** (Preprocessing of mzML/mzXML input files and discarding low-quality MS/MS spectra in the ZODIAC workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **PubChem** (PUG REST fastformula endpoint queried to count compound identifiers per novel formula, verifying database absence)
- **ChemSpider** (Second public structure database for absence checks; requires account credentials, so deferred to expert review)

## Examples

```
curl https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/fastformula/C24H47BrNO8P/cids/JSON
```

## Evaluation signals

- Verification CSV contains exactly 62 data rows (59 diatoms + 3 tomato formulas), matching Supplementary Tables 3 and 4.
- Every PubChem fastformula query returns a compound identifier count of 0, consistent with the claim that these formulas are absent from public databases.
- Each nominated formula satisfies all three strict filters: ZODIAC score >= 0.999, >= 95% MS/MS intensity explained by SIRIUS, and a similarity-network connection to >= 20 compounds.
- Formula strings normalize to Hill notation before database queries.
- No MS1 peaks at m/z +18.01 (water loss), +43.99 (CO2 loss), or -17.03 (ammonium adduct), and no database formulas corresponding to the novel formula ± H2O/CO2/NH3 — ruling out adduct/neutral-loss artifacts.

## Limitations

- Reproducibility requires the modified OpenMS 2.4.0 plus a separate patch file rather than stock OpenMS.
- The ChemSpider absence check needs API credentials unavailable to an automated agent and may have to be deferred to expert review.
- A complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible.
- SIRIUS performance drops substantially for compounds above 500 Da — the regime where most novel formulas arise (e.g. compounds above 700 Dalton in the plant extract).
- Structure databases will always return incorrect hits for compounds with novel molecular formulas, often without being noticed.

## Evidence

- [readme] SIRIUS framework description: [evidence span withheld — anchor status: external_doc]
