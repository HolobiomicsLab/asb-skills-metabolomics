---
name: fragmentation-tree-intensity-explanation-analysis
description: Use when reproducing the preprocessing filter that reduced raw feature
  tables (mzML/mzXML-derived) before ZODIAC molecular formula annotation, and when
  validating the reported per-dataset feature counts for the five datasets.
when_to_use_negative:
- Input is already a post-filter feature table that ZODIAC consumed directly (e.g.,
  the 784 dendroides or 2,584 tomato features) — re-filtering would not reproduce
  raw counts
- The goal is ZODIAC's post-annotation novelty filters (ZODIAC score >= 0.999, >=
  95% intensity explained, network connectivity >= 20) — those operate on formula
  annotations, not on input spectra
- Datasets outside the five studied here, since the validation thresholds and reported
  counts are dataset-specific
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: OpenMS
  role: Processes the input mzML/mzXML files and implements the low-quality MS/MS
    spectra discarding step; the paper used a modified OpenMS 2.4.0 with the ZODIAC
    workflow changes
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: SIRIUS
  role: Computes fragmentation trees for the top-50 molecular formula candidates per
    feature; candidate trees must explain at least 5 peaks covering >= 80% of total
    MS/MS intensity for a feature to survive
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS 2.4.0
  role: Specific version whose modified build (with patch file from the data package)
    is required to reproduce the exact preprocessing
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:b8d9d248458ea96cf2e1fca3cb9e8a09692986b8e04505e7825ac1ed4edfb6b9
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fragmentation-tree-intensity-explanation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the low-quality MS/MS feature filtering step used before ZODIAC analysis: merging fragment peaks, discarding features with too few peaks, and retaining only features whose SIRIUS fragmentation trees (top-50 molecular formula candidates) explain sufficient MS/MS intensity, then verifying per-dataset before/after feature counts.

## When to use

Use when reproducing the preprocessing filter that reduced raw feature tables (mzML/mzXML-derived) before ZODIAC molecular formula annotation, and when validating the reported per-dataset feature counts for the five datasets.

## When NOT to use

- Input is already a post-filter feature table that ZODIAC consumed directly (e.g., the 784 dendroides or 2,584 tomato features) — re-filtering would not reproduce raw counts
- The goal is ZODIAC's post-annotation novelty filters (ZODIAC score >= 0.999, >= 95% intensity explained, network connectivity >= 20) — those operate on formula annotations, not on input spectra
- Datasets outside the five studied here, since the validation thresholds and reported counts are dataset-specific

## Inputs

- Raw mzML/mzXML files or pre-filter feature tables for the five datasets (dendroides, NIST1950, tomato, diatoms, mice stool)
- Virtual machine / data package from https://bio.informatik.uni-jena.de/data/
- Modified OpenMS 2.4.0 build (branch release_2.4.0_modifications_ZODIAC_workflow) plus patch file

## Outputs

- Filtered feature table per dataset (low-quality MS/MS features discarded)
- before_after_feature_counts.csv with exactly 5 rows (columns: dataset, features_before, features_after)

## How to apply

Download the deposited virtual machine/data package from https://bio.informatik.uni-jena.de/data/ containing the pre-filter feature tables for the five datasets (dendroides, NIST1950, tomato, diatoms, mice stool). For each dataset, merge fragment peaks within a 10 ppm / 0.0025 m/z tolerance and discard features left with fewer than 5 fragment peaks after merging. For surviving features, run SIRIUS to compute fragmentation trees for the top-50 molecular formula candidates and discard any feature for which no candidate tree explains at least 5 peaks covering >= 80% of the total MS/MS intensity. Record feature counts before and after filtering per dataset. Note that reproducibility requires the modified OpenMS 2.4.0 branch (release_2.4.0_modifications_ZODIAC_workflow) plus a patch file, not stock OpenMS.

## Related tools

- **OpenMS** (Processes the input mzML/mzXML files and implements the low-quality MS/MS spectra discarding step; the paper used a modified OpenMS 2.4.0 with the ZODIAC workflow changes) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **SIRIUS** (Computes fragmentation trees for the top-50 molecular formula candidates per feature; candidate trees must explain at least 5 peaks covering >= 80% of total MS/MS intensity for a feature to survive) — https://github.com/sirius-ms/sirius
- **OpenMS 2.4.0** (Specific version whose modified build (with patch file from the data package) is required to reproduce the exact preprocessing) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- before_after_feature_counts.csv reproduces the reported counts exactly: dendroides 1,078 to 784, NIST1950 568 to 400, tomato 3,583 to 2,584, diatoms 3,227 to 2,075, mice stool 577 to 377
- Output CSV has exactly 5 rows with columns dataset, features_before, features_after
- Every discarded feature either had fewer than 5 fragment peaks after 10 ppm / 0.0025 m/z merging, or had no top-50 candidate fragmentation tree explaining at least 5 peaks covering >= 80% of MS/MS intensity
- Every surviving feature satisfies both the fragment-peak minimum and the fragmentation-tree intensity-explanation threshold
- Filtering is documented against Supplementary Fig. 7, Supplementary Table 2 and 7, and the Materials & Methods section

## Limitations

- Stock OpenMS is insufficient: reproducibility requires a modified version of OpenMS 2.4.0 plus a separate patch file available from https://bio.informatik.uni-jena/data/ (branch release_2.4.0_modifications_ZODIAC_workflow)
- Mice stool spectra were measured with a broader isolation window, so the dataset's fragmentation spectra may behave differently under the same thresholds
- The paper states high-quality adduct annotations should be established during preprocessing, which this filtering step alone does not guarantee

## Evidence

- [results] OpenMS processing of raw files: [evidence span withheld — anchor status: not_found]
- [discussion] Modified OpenMS requirement: [evidence span withheld — anchor status: not_found]
