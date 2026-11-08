---
name: proteomic-metabolomic-benchmark-reproduction
description: Use when benchmarking peak-list alignment algorithms on LC-MS peak lists
  with known ground-truth alignments, or when reproducing the reported RANSAC vs Join
  aligner precision/recall comparison.
when_to_use_negative:
- Evaluating alignment on synthetic datasets only — the paper separately reports 100%
  precision and recall for RANSAC on the 12 synthetic lipidomic datasets, which is
  a different benchmark
- The goal is general MZmine 2 data processing (peak picking, deisotoping, gap filling,
  normalization) rather than aligner benchmarking against ground truth
- Ground-truth alignment tables are unavailable for the input datasets, making precision/recall
  scoring impossible
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Platform in which both the RANSAC aligner and Join aligner are run over the
    imported peak lists at fixed m/z tolerance and RT tolerances of 5 s, 20 s, and
    50 s
  repo: https://github.com/mzmine/mzmine2
- name: RANSAC aligner
  role: Alignment method under test; iteratively fits alignment functions to matching
    peak pairs and discards non-matching outliers
- name: Join aligner
  role: Baseline alignment method against which RANSAC performance is compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:602b8fcfb80451f935d495826afc6762d46dc74fc0da0dc5dac1550d2eb3e09c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Join aligner
  - MZmine 2
  - RANSAC aligner
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# proteomic-metabolomic-benchmark-reproduction

## Summary

Reproduces the MZmine 2 benchmark comparing the RANSAC aligner against the Join aligner on the real proteomic (P1, P2) and metabolomic (M1, M2) datasets from Lange et al., scoring precision and recall against ground-truth alignments.

## When to use

Use when benchmarking peak-list alignment algorithms on LC-MS peak lists with known ground-truth alignments, or when reproducing the reported RANSAC vs Join aligner precision/recall comparison.

## When NOT to use

- Evaluating alignment on synthetic datasets only — the paper separately reports 100% precision and recall for RANSAC on the 12 synthetic lipidomic datasets, which is a different benchmark
- The goal is general MZmine 2 data processing (peak picking, deisotoping, gap filling, normalization) rather than aligner benchmarking against ground truth
- Ground-truth alignment tables are unavailable for the input datasets, making precision/recall scoring impossible

## Inputs

- Real proteomic peak-list datasets P1 and P2 (with fractions) from Lange et al.
- Real metabolomic peak-list datasets M1 and M2 from Lange et al.
- Tables of 'ground truth' alignments for each dataset
- Peak lists importable into MZmine 2

## Outputs

- Precision and recall values per dataset, fraction, aligner, and RT tolerance (5 s, 20 s, 50 s)
- Comparison table of precision/recall across the 13 pairwise alignments for RANSAC vs Join aligner
- Verification that RANSAC was better in 11 of 13 alignments, best on all P1 fractions, and worse only on P2 fraction 00

## How to apply

Obtain the real proteomic (P1, P2) and metabolomic (M1, M2) datasets from Lange et al. together with their tables of 'ground truth' alignments, and import each peak list into MZmine 2 preserving the ground-truth labels for scoring. Run the Join aligner over every dataset and fraction at RT tolerances of 5 s, 20 s, and 50 s with m/z tolerance held fixed, then run the RANSAC aligner on identical peak lists at the same RT tolerance settings, relying on its iterative model fitting to fit alignment functions to matching peak pairs and discard non-matching outliers. Score each aligned peak list against the ground-truth alignments, computing precision (fraction of aligned peak pairs that are correct) and recall (fraction of true peak pairs that are aligned) per dataset, fraction, aligner, and RT tolerance. Assemble the values into a comparison table covering the 13 pairwise alignments (P1 and P2 fractions, M1, M2) for both aligners and verify the expected pattern: RANSAC better in 11 of 13 alignments, best on all P1 fractions, worse only on P2 dataset fraction 00.

## Related tools

- **MZmine 2** (Platform in which both the RANSAC aligner and Join aligner are run over the imported peak lists at fixed m/z tolerance and RT tolerances of 5 s, 20 s, and 50 s) — https://github.com/mzmine/mzmine2
- **RANSAC aligner** (Alignment method under test; iteratively fits alignment functions to matching peak pairs and discards non-matching outliers)
- **Join aligner** (Baseline alignment method against which RANSAC performance is compared)

## Evaluation signals

- RANSAC aligner outperforms the Join aligner in 11 of 13 pairwise alignments
- RANSAC gives worse results in exactly one case: the P2 dataset fraction 00 (which has over 6800 rows after alignment)
- RANSAC is best on all P1 fractions
- Precision is computed as the fraction of aligned peak pairs that are correct and recall as the fraction of true peak pairs that are aligned, for each RT tolerance (5 s, 20 s, 50 s)
- m/z tolerance is held fixed across both aligners so only the RT tolerance varies

## Limitations

- The RANSAC aligner underperforms the Join aligner on the P2 dataset fraction 00, attributed to the high number of features in this fraction (over 6800 rows after alignment)
- MZmine 2 modules were tested mainly on LC-MS data, so extension to other data types is not demonstrated
- The Tranche hash for downloading the associated data is referenced but not present in the text, complicating raw data retrieval

## Evidence

- [results] to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [results] RANSAC aligner provided better results in 11 of 13 alignments compared to Join aligner, worse in only one case (P2 dataset fraction 00): "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] the high number of features in this fraction (over 6800 rows after alignment): "the high number of features in this fraction (over 6800 rows after alignment)"
- [supplementary] Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets.: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [readme] All development happens in a new repository: [mzmine GitHub](https://github.com/mzmine/mzmine3): "This repository is only there as a reference to the old MZmine 2 that was succeeded by mzmine."
