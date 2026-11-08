---
name: feature-count-reconciliation-across-datasets
description: Use when raw mzML/mzXML feature tables must be quality-filtered and the
  reported before/after feature counts (e.g. dendroides 1,078→784) need to be reproduced
  or audited.
when_to_use_negative:
- The input is already a post-filter feature table from the deposited data package
  with no raw mzML/mzXML spectra to re-filter
- The goal is candidate-level filtering after ZODIAC scoring (e.g. ZODIAC score >=
  0.999, >= 95% explained intensity) rather than preprocessing-level low-quality feature
  removal
- The datasets are not among the five ZODIAC evaluation datasets, so no published
  before/after counts exist for validation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: OpenMS
  role: Processes the input mzML/mzXML files and performs the low-quality MS/MS spectra
    filtering during preprocessing
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: OpenMS 2.4.0
  role: The specific modified version used in the ZODIAC workflow; a patch file with
    the changes is available from the Jena data page
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: SIRIUS
  role: Computes fragmentation trees for the top-50 molecular formula candidates to
    decide whether a feature's MS/MS spectrum explains enough peaks and intensity
    to survive filtering
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:97ccb8a4b323fbc623bfe1d0143b3e478e9640a53832c0381e72dde31c0a8ab9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS
  - OpenMS 2.4.0
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# feature-count-reconciliation-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the low-quality MS/MS feature filtering step of the ZODIAC preprocessing workflow (OpenMS peak merging, fragment-peak threshold, SIRIUS fragmentation-tree intensity coverage) and reconciles the resulting before/after feature counts against the five published datasets.

## When to use

Use when raw mzML/mzXML feature tables must be quality-filtered and the reported before/after feature counts (e.g. dendroides 1,078→784) need to be reproduced or audited.

## When NOT to use

- The input is already a post-filter feature table from the deposited data package with no raw mzML/mzXML spectra to re-filter
- The goal is candidate-level filtering after ZODIAC scoring (e.g. ZODIAC score >= 0.999, >= 95% explained intensity) rather than preprocessing-level low-quality feature removal
- The datasets are not among the five ZODIAC evaluation datasets, so no published before/after counts exist for validation

## Inputs

- Raw or pre-filter feature tables (mzML/mzXML-derived) for the five datasets: dendroides, NIST1950, tomato, diatoms, mice stool
- Data package / virtual machine from https://bio.informatik.uni-jena.de/data/ containing the ZODIAC workflow inputs
- Modified OpenMS 2.4.0 build (or patch file) from https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Outputs

- before_after_feature_counts.csv with exactly 5 rows and columns: dataset, features_before, features_after
- Per-dataset filtered feature lists that pass the fragment-peak and SIRIUS tree-coverage criteria

## How to apply

Download the deposited virtual machine / data package from https://bio.informatik.uni-jena.de/data/ containing pre-filter feature tables for the five datasets (dendroides, NIST1950, tomato, diatoms, mice stool), and use the modified OpenMS 2.4.0 from the marcus-ludwig fork rather than stock OpenMS. For each dataset, merge fragment peaks within a 10 ppm / 0.0025 m/z tolerance and discard features left with fewer than 5 fragment peaks after merging. For surviving features, use SIRIUS to compute fragmentation trees for the top-50 molecular formula candidates and discard any feature for which no candidate tree explains at least 5 peaks covering >= 80% of the total MS/MS intensity. Record feature counts before and after filtering per dataset and write them to before_after_feature_counts.csv with columns (dataset, features_before, features_after); correctness is judged by exact reproduction of the reported counts. The rationale is that the article documents this filtering procedure in Supplementary Fig. 7, Supplementary Table 2 and 7, and the Materials & Methods, so any reproduction must match those numbers exactly.

## Related tools

- **OpenMS** (Processes the input mzML/mzXML files and performs the low-quality MS/MS spectra filtering during preprocessing) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **OpenMS 2.4.0** (The specific modified version used in the ZODIAC workflow; a patch file with the changes is available from the Jena data page) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **SIRIUS** (Computes fragmentation trees for the top-50 molecular formula candidates to decide whether a feature's MS/MS spectrum explains enough peaks and intensity to survive filtering) — https://github.com/sirius-ms/sirius

## Evaluation signals

- before_after_feature_counts.csv contains exactly 5 rows with columns (dataset, features_before, features_after)
- Reproduced counts match the published values exactly: dendroides 1,078→784, NIST1950 568→400, tomato 3,583→2,584, diatoms 3,227→2,075, mice stool 577→377
- features_after < features_before for every dataset, with retention rates in the ~53–70% range consistent with the reported figures
- Every retained feature has at least 5 merged fragment peaks (10 ppm / 0.0025 m/z tolerance) and at least one of its top-50 SIRIUS candidate trees explaining >= 5 peaks and >= 80% of total MS/MS intensity

## Limitations

- Reproduction requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- The filtering procedure is documented across Supplementary Fig. 7, Supplementary Table 2 and 7, and the Materials & Methods; omitting any of these sources can cause silent parameter mismatches
- Validation is exact-count matching against five fixed datasets, so the reconstruction cannot be evaluated on data outside those datasets

## Evidence

- [results] OpenMS preprocessing: [evidence span withheld — anchor status: not_found]
- [discussion] Modified OpenMS requirement: [evidence span withheld — anchor status: not_found]
