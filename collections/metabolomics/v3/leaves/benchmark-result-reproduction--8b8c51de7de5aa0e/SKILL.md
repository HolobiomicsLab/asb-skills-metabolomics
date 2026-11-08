---
name: benchmark-result-reproduction
description: Use when reproducing published molecular-formula or structure-identification
  benchmark results from LC-MS/MS challenge data using SIRIUS 4 with CSI:FingerID
  and isotope-pattern scoring.
when_to_use_negative:
- The goal is de novo formula/structure identification on new samples rather than
  reproducing a published benchmark's identification rates.
- The data are not LC-MS/MS challenge spectra with known reference formulas/structures,
  so top-k identification rates cannot be defined.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 4
  role: Performs isotope pattern analysis to rank candidate molecular formulas and
    computes fragmentation trees from the MS/MS spectra
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Receives fragmentation trees and spectra from SIRIUS via web service upload
    and returns ranked candidate structures from molecular structure databases per
    challenge
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Command-line/GUI framework that seamlessly integrates CSI:FingerID, CANOPUS
    and MSNovelist web services for the benchmark reproduction run
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:5d434b4c461fee2a2e2d205d870a91f0fba4e7e8689a9bff4fbe4e976a4e84a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-result-reproduction

## Summary

Reproduces published benchmark results of SIRIUS 4 / CSI:FingerID by re-running formula and structure identification on a challenge dataset (e.g. CASMI 2016) and recomputing top-k identification rates. Validates that the SIRIUS 4 pipeline reproduces the reported rates, such as 93.8% correct molecular formulas at top 1.

## When to use

Use when reproducing published molecular-formula or structure-identification benchmark results from LC-MS/MS challenge data using SIRIUS 4 with CSI:FingerID and isotope-pattern scoring.

## When NOT to use

- The goal is de novo formula/structure identification on new samples rather than reproducing a published benchmark's identification rates.
- The data are not LC-MS/MS challenge spectra with known reference formulas/structures, so top-k identification rates cannot be defined.

## Inputs

- CC-BY CASMI 2016 re-evaluation LC-MS/MS challenge spectra (per-challenge MS/MS and isotope pattern data, positive and negative ionization modes)
- Reference molecular formulas and reference structures for each challenge
- CSI:FingerID predictor training structures (positive and negative ion mode via the trainingstructures API)
- SIRIUS 4 fragmentation trees and spectra generated from the input challenges

## Outputs

- Ranked candidate molecular formulas per challenge (top-2)
- Ranked candidate structures per challenge (top-10) and top-3 within the biomolecular database
- CASMI 2016 results table (casmi2016_results_table.csv) with top-1 and top-k identification rates per ionization mode and per database
- Per-challenge formulas/structures labelled correct vs incorrect at top-k

## How to apply

Download the CC-BY CASMI 2016 re-evaluation LC-MS/MS challenge data from https://bio.informatik.uni-jena.de/data, extracting per-challenge MS/MS spectra, isotope patterns, and reference formulas/structures. Run SIRIUS 4 isotope pattern analysis to rank candidate molecular formulas from the high-resolution mass spectra, and compute fragmentation trees from the MS/MS fragmentation patterns. Upload fragmentation trees and spectra to CSI:FingerID to search molecular structure databases and retrieve ranked candidate structures per challenge, optionally restricting to the biomolecular structure database for positive-mode challenges. Exclude challenges whose correct structure is not structure-disjoint from CSI:FingerID's training data (available via the trainingstructures API endpoints for predictor=1 and predictor=2), then compute top-1 and top-k identification rates — top 2 for formulas, top 10 for structures, top 3 for the biomolecular database. Assemble per-mode and per-database rates into a results table and compare against the published rates (e.g. 93.8% correct molecular formula at top 1 and 96.2% at top 2 using MS/MS and isotope pattern data) within rounding tolerance.

## Related tools

- **SIRIUS 4** (Performs isotope pattern analysis to rank candidate molecular formulas and computes fragmentation trees from the MS/MS spectra) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Receives fragmentation trees and spectra from SIRIUS via web service upload and returns ranked candidate structures from molecular structure databases per challenge) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Command-line/GUI framework that seamlessly integrates CSI:FingerID, CANOPUS and MSNovelist web services for the benchmark reproduction run) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input casmi2016_spectra.mgf --output casmi2016_results --project casmi2016; curl -X POST "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/search" -d @casmi2016_results
```

## Evaluation signals

- Top-1 correct molecular formula rate reproduces the published 93.8% (MS/MS + isotope pattern data) within rounding tolerance
- Top-2 correct molecular formula rate reproduces the published 96.2% within rounding tolerance
- Structure identification rates exceed 70% on the challenging metabolomics datasets, matching the reported SIRIUS 4 performance
- Per-challenge correct/incorrect labels agree with the reference formulas and structures extracted from the CASMI 2016 data
- Output CSV matches the expected schema of casmi2016_results_table.csv with per-mode and per-database rates

## Limitations

- Challenges whose correct structure is not structure-disjoint from CSI:FingerID's training data must be excluded from the rate computation.
- Isotope pattern analysis is a rewritten implementation whose stability is stated only as 'hopefully more stable than before'; edge cases may still fail.
- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are for academic research and education use only — redistribution for commercial purposes is prohibited.
- Reproduction depends on the exact SIRIUS 4 version and predictor releases referenced in the paper; results may shift with newer releases (e.g. v6.3.7).

## Evidence

- [results] 93.8% top-1 formula rate: [evidence span withheld — anchor status: not_found]
- [methods] fragmentation trees and spectra upload to CSI:FingerID: "Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services."
- [abstract] identification rate threshold: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [methods] isotope pattern ranking of formulas: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] academic-only web services: "The SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist and others) hosted by the [Böcker group](https://bio.informatik.uni-jena.de/) are for academic research and education use only."
