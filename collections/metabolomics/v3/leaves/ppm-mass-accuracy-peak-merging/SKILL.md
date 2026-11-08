---
name: ppm-mass-accuracy-peak-merging
description: Use when raw mzML/mzXML LC-MS/MS files must be preprocessed into feature
  tables with clean, merged MS/MS fragment peak lists (e.g., before computing fragmentation
  trees with SIRIUS or reranking with ZODIAC).
when_to_use_negative:
- Input is already a filtered feature table — the before/after counting logic expects
  pre-filter feature counts (e.g., dendroides 1,078 to 784)
- Goal is reranking SIRIUS molecular formula candidates with ZODIAC's Gibbs sampling
  rather than the preprocessing/peak-merging stage itself
- Goal is ZODIAC post-hoc filters such as minimum ZODIAC score 0.999 or 95% explained
  MS/MS intensity, which operate on ZODIAC output, not on raw spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Modified OpenMS 2.4.0 processes the input mzML/mzXML files, merges fragment
    peaks under the 10 ppm / 0.0025 m/z tolerance, and discards low-quality MS/MS
    spectra
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:b5b3a89047985ba06497dd23c00e4391928087e85b6b35e43a67756309a991b7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ppm-mass-accuracy-peak-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges fragment peaks belonging to the same feature within a 10 ppm / 0.0025 m/z mass-accuracy window as part of the OpenMS-based preprocessing that discards low-quality MS/MS spectra before SIRIUS/ZODIAC molecular formula annotation.

## When to use

Use when raw mzML/mzXML LC-MS/MS files must be preprocessed into feature tables with clean, merged MS/MS fragment peak lists (e.g., before computing fragmentation trees with SIRIUS or reranking with ZODIAC).

## When NOT to use

- Input is already a filtered feature table — the before/after counting logic expects pre-filter feature counts (e.g., dendroides 1,078 to 784)
- Goal is reranking SIRIUS molecular formula candidates with ZODIAC's Gibbs sampling rather than the preprocessing/peak-merging stage itself
- Goal is ZODIAC post-hoc filters such as minimum ZODIAC score 0.999 or 95% explained MS/MS intensity, which operate on ZODIAC output, not on raw spectra

## Inputs

- Raw LC-MS/MS mzML or mzXML files (e.g., from the dendroides, NIST1950, tomato, diatoms, or mice stool datasets)
- Feature tables produced by OpenMS feature detection with attached MS/MS fragment peak lists

## Outputs

- Filtered feature tables containing only features with >= 5 merged fragment peaks
- before_after_feature_counts.csv with 5 rows (columns: dataset, features_before, features_after)

## How to apply

Process the input mzML/mzXML files with the modified OpenMS 2.4.0 build used in the ZODIAC workflow, which discards low-quality MS/MS spectra during loading. For each detected feature, merge fragment peaks that fall within 10 ppm (or 0.0025 m/z absolute tolerance) so that replicate/noise peaks of the same fragment collapse into single peaks. Discard any feature left with fewer than 5 fragment peaks after merging, since such spectra are too sparse for fragmentation tree computation. For surviving features, SIRIUS should then be able to compute fragmentation trees for the top-50 molecular formula candidates, and features whose candidate trees cannot explain sufficient peaks are removed. The rationale is that ZODIAC quality depends on high-quality adduct and peak annotations established during preprocessing, not on later correction from isotope or MS/MS data.

## Related tools

- **OpenMS** (Modified OpenMS 2.4.0 processes the input mzML/mzXML files, merges fragment peaks under the 10 ppm / 0.0025 m/z tolerance, and discards low-quality MS/MS spectra) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Reconstructed before/after feature counts must exactly reproduce the reported values: dendroides 1,078 to 784, NIST1950 568 to 400, tomato 3,583 to 2,584, diatoms 3,227 to 2,075, mice stool 577 to 377
- before_after_feature_counts.csv contains exactly 5 rows with columns dataset, features_before, features_after
- Every retained feature has at least 5 fragment peaks after merging under the 10 ppm / 0.0025 m/z tolerance
- Downstream SIRIUS fragmentation tree computation succeeds on the merged peak lists for the top-50 candidate molecular formulas
- The OpenMS version used is the modified 2.4.0 ZODIAC-workflow build, not stock OpenMS

## Limitations

- Reproducibility requires the modified OpenMS 2.4.0 from GitHub plus a separate patch file (available from https://bio.informatik.uni-jena.de/data/), not stock OpenMS
- The article documents the filtering only at a high level ('low-quality MS/MS spectra were discarded'); precise merge thresholds are recovered from the deposited workflow/virtual machine
- Preprocessed results depend on the specific tolerance parameters; the mice stool dataset used a broader MS/MS isolation window, which may affect fragment peak lists

## Evidence

- [results] openms_discards_low_quality: [evidence span withheld — anchor status: not_found]
- [discussion] modified_openms_github: [evidence span withheld — anchor status: not_found]
