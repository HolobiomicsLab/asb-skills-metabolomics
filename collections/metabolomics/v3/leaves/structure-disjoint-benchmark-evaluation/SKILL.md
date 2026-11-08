---
name: structure-disjoint-benchmark-evaluation
description: Use when re-evaluating formula or structure identification tools (e.g.
  SIRIUS 4 / CSI:FingerID) on a public challenge dataset such as CASMI 2016 and a
  fair, training-independent estimate of top-1/top-k identification rates is needed.
when_to_use_negative:
- When the correct answer for some challenge compounds lies inside the tool's training
  set and structure-disjoint filtering is not applied — rates will be inflated and
  non-comparable
- When the dataset is not a curated challenge/benchmark with known reference structures
  (e.g. routine exploratory analysis of unidentified metabolites)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: 'Tool under test: deduces molecular formulas by ranking isotope patterns from
    high-resolution mass spectra and computes fragmentation trees for each challenge
    spectrum'
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Structure search service to which fragmentation trees and spectra are directly
    uploaded from SIRIUS to retrieve ranked candidate structures per challenge; its
    training structures define the structure-disjoint filter
  repo: https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:be5ee582fbc21db6b78d5f93f4d67086b11e0775f6ec84e33adb651c095ffe9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# structure-disjoint-benchmark-evaluation

## Summary

An evaluation skill for benchmarking molecular formula and structure identification tools on curated challenge datasets (e.g. CASMI 2016), computing top-1 and top-k identification rates while excluding compounds whose reference structures are not structure-disjoint from the tool's training data to avoid inflated performance estimates.

## When to use

Use when re-evaluating formula or structure identification tools (e.g. SIRIUS 4 / CSI:FingerID) on a public challenge dataset such as CASMI 2016 and a fair, training-independent estimate of top-1/top-k identification rates is needed.

## When NOT to use

- When the correct answer for some challenge compounds lies inside the tool's training set and structure-disjoint filtering is not applied — rates will be inflated and non-comparable
- When the dataset is not a curated challenge/benchmark with known reference structures (e.g. routine exploratory analysis of unidentified metabolites)

## Inputs

- CC-BY CASMI 2016 re-evaluation LC-MS/MS challenge data with spectra, adduct/ion-mode annotations, and reference molecular formulas and structures
- Tool-under-test candidate rankings (molecular formula candidates from isotope pattern/fragmentation tree analysis; structure candidates from a structure-database search)
- Tool training-data structure list (e.g. CSI:FingerID training structures per ionization mode) for computing structure-disjoint filtering

## Outputs

- Per-challenge top-k identification outcomes (top 2 for formulas, top 10 for structures, top 3 for database-restricted searches)
- Aggregate identification rates per ionization mode and per database
- Results summary table (e.g. casmi2016_results_table.csv)

## How to apply

Acquire the benchmark challenge data (e.g. the CC-BY CASMI 2016 re-evaluation LC-MS/MS data from https://bio.informatik.uni-jena.de/data), extract each challenge's MS/MS spectrum together with its reference molecular formula and structure. Run the tool under test on all challenges, ranking molecular formula candidates via isotope pattern analysis and fragmentation tree computation in both ionization modes, then submit fragmentation trees and spectra to the structure search service (e.g. CSI:FingerID) to rank candidate structures. Optionally restrict the search to a specific database (e.g. a biomolecular structure database for positive-mode challenges). Critically, exclude every challenge whose correct reference structure is not structure-disjoint from the tool's training data — for CSI:FingerID these training structures are retrievable via its web API — so that compounds the predictor was trained on cannot inflate the score. Compute top-k identification rates (top 2 for formulas, top 10 for structures, top 3 for a database-restricted search) and assemble the per-mode and per-database results into a summary table (e.g. casmi2016_results_table.csv), checking each recomputed rate against the published values within rounding tolerance.

## Related tools

- **SIRIUS 4** (Tool under test: deduces molecular formulas by ranking isotope patterns from high-resolution mass spectra and computes fragmentation trees for each challenge spectrum) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Structure search service to which fragmentation trees and spectra are directly uploaded from SIRIUS to retrieve ranked candidate structures per challenge; its training structures define the structure-disjoint filter) — https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1

## Evaluation signals

- Recomputed top-2 molecular formula identification rate (~96% on CASMI 2016 with MS/MS + isotope pattern data) agrees with the published re-evaluation value within rounding tolerance
- Recomputed top-1 formula identification rate (~94%) agrees with the published value
- Structure identification rates exceed 70% on challenging metabolomics datasets, matching the paper's reported threshold
- Every included challenge's reference structure is verified structure-disjoint from CSI:FingerID training data (predictor=1 positive / predictor=2 negative sets)
- Results table rows cover per-ionization-mode and per-database rates as separate aggregates

## Limitations

- Structure-disjoint filtering removes challenges whose correct structures were seen in training, reducing effective sample size and possibly biasing toward harder compounds
- Benchmark validity depends on reference structures and annotations in the public dataset; any misannotation propagates into the evaluation
- Isotope pattern analysis in the released tool was rewritten with stability stated only as 'hopefully more stable than before' in the changelog

## Evidence

- [results] SIRIUS 4 assigned the correct molecular formula to 93.8% of the compounds using MS/MS and isotope pattern data, and 96.2% in the top 2.: [evidence span withheld — anchor status: not_found]
- [methods] Exclude challenges whose correct structure is not structure-disjoint from CSI:FingerID's training data, then compute top-1 and top-k identification rates (top 2 for formulas, top 10 for structures, top 3 for the biomolecular database).: [evidence span withheld — anchor status: not_found]
- [abstract] Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets.: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [methods] Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services.: "Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services."
- [discussion] isotope pattern analysis is now rewritten and hopefully more stable than before: "isotope pattern analysis is now rewritten and hopefully more stable than before"
