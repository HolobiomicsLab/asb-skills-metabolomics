---
name: msms-dataset-batch-processing
description: Use when benchmarking formula assignment across multiple qualified MS/MS
  datasets — e.g. computing rank#1/top-3/top-5 accuracy and total processing time
  for SIRIUS, Buddy, or FSA on identical spectra and hardware.
when_to_use_negative:
- The target compounds contain elements beyond C, H, O, N, S, P, or depend on adduct-specific
  chemical rules — FSA has not been validated in these cases
- The goal is structural elucidation or spectral library matching rather than fast
  formula prescreening — FSA does not use structural information and its rank#1 accuracy
  is not comparable to Buddy/SIRIUS
- No ground-truth reference formulas are available for the datasets, making rank-window
  accuracy computation impossible
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Target formula-ranking tool under evaluation; ranks candidate formulas by
    fragment subset analysis and provides the runtime/accuracy baseline
  repo: kslynn128171/FSA
- name: SIRIUS
  role: State-of-the-art baseline; run at version 5.8.6 with the paper's optimal ppm
    tolerance to produce ranked candidate formulas per spectrum
- name: Buddy
  role: State-of-the-art baseline; run at version 0.3.8 with 10–20 ppm mass tolerance
    to produce ranked candidate formulas per spectrum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:33b069cb353315131c4ea6c18dc883da947a6b40be302c5f1c74c8ab45d63052
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - Buddy
  - FSA
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msms-dataset-batch-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-process multiple MS/MS datasets through formula-ranking tools (FSA, SIRIUS 5.8.6, Buddy 0.3.8), recording per-dataset rank-1/top-3/top-5 accuracy and wall-clock runtime to reproduce the published head-to-head comparison.

## When to use

Use when benchmarking formula assignment across multiple qualified MS/MS datasets — e.g. computing rank#1/top-3/top-5 accuracy and total processing time for SIRIUS, Buddy, or FSA on identical spectra and hardware.

## When NOT to use

- The target compounds contain elements beyond C, H, O, N, S, P, or depend on adduct-specific chemical rules — FSA has not been validated in these cases
- The goal is structural elucidation or spectral library matching rather than fast formula prescreening — FSA does not use structural information and its rank#1 accuracy is not comparable to Buddy/SIRIUS
- No ground-truth reference formulas are available for the datasets, making rank-window accuracy computation impossible

## Inputs

- Eight qualified LC-MS/MS test datasets with MS/MS spectra from diverse sample types
- Ground-truth reference molecular formulas per spectrum/metabolite
- Tool binaries or containers: SIRIUS 5.8.6, Buddy 0.3.8, FSA (kslynn128171/FSA)
- Mass tolerance settings per tool (paper's optimal ppm for SIRIUS; 10–20 ppm for Buddy)

## Outputs

- Per-dataset rank-1, top-3, and top-5 accuracy values for each tool
- Total and per-spectrum wall-clock processing times on identical hardware
- A comparison table of accuracy and runtime alongside FSA's published values
- Computed FSA-vs-tool speed ratios (~148x vs Buddy, ~134x vs SIRIUS)

## How to apply

Load the eight qualified MS/MS datasets together with each dataset's ground-truth reference formulas. Run SIRIUS 5.8.6 with the paper's optimal ppm mass tolerance and Buddy 0.3.8 with a 10–20 ppm mass tolerance on every dataset, producing ranked candidate molecular formulas per spectrum. For each tool, compute rank-1, top-3, and top-5 accuracy per dataset by checking whether the correct formula falls within the corresponding rank window. Record total and per-spectrum wall-clock time on the same hardware; FSA's expected throughput is ~0.024 s per spectrum (12.5 min for the eight-dataset suite vs 1848 min for Buddy and 1678 min for SIRIUS). Assemble a comparison table and verify the reproduced FSA-vs-tool speed ratios (~148x over Buddy, ~134x over SIRIUS) and that per-dataset accuracies match the published Table 3 values.

## Related tools

- **FSA** (Target formula-ranking tool under evaluation; ranks candidate formulas by fragment subset analysis and provides the runtime/accuracy baseline) — kslynn128171/FSA
- **SIRIUS** (State-of-the-art baseline; run at version 5.8.6 with the paper's optimal ppm tolerance to produce ranked candidate formulas per spectrum)
- **Buddy** (State-of-the-art baseline; run at version 0.3.8 with 10–20 ppm mass tolerance to produce ranked candidate formulas per spectrum)

## Evaluation signals

- Per-dataset rank-1/top-3/top-5 accuracies for SIRIUS and Buddy match the published Table 3 values within the paper's reporting granularity
- Computed speed ratios reproduce ~148x (FSA vs Buddy) and ~134x (FSA vs SIRIUS): 12.5 min vs 1848 min and 1678 min respectively
- FSA's aggregate accuracy reaches 85.28% rank#1 and 97.35% top-5 over 30,690 MS/MS spectra from 6792 metabolites
- FSA's average per-spectrum processing time is approximately 0.024 s
- All tools run on the same eight datasets and the same hardware, so runtime entries are directly comparable

## Limitations

- FSA's rank#1 accuracy (74.01% in the head-to-head; Buddy 97.63%, SIRIUS 70.31% across the eight datasets) is not yet comparable to state-of-the-art tools because it does not use structural information
- FSA has only been validated on compounds composed of C, H, O, N, S, and P; adduct-type chemical rules are not yet incorporated
- The bundled PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), which may affect candidate formula coverage
- Few chemical rules are implemented to rank possible formulas and filter out unfeasible ones, causing ambiguity in certain compound species

## Evidence
