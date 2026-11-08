---
name: precision-recall-computation
description: Use when evaluating peak list alignment quality against known ground-truth
  alignments, e.g. comparing aligners (RANSAC vs Join) across LC-MS fractions at several
  RT tolerances.
when_to_use_negative:
- The peak lists have no ground-truth alignment tables, so precision/recall cannot
  be scored against reference pairings
- The goal is de novo detection or gap filling of features rather than judging the
  correctness of an existing alignment
- Only one aligner is being run with no comparison baseline — precision/recall vs
  ground truth is chiefly a comparative benchmark here
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Platform in which peak lists are imported and the Join and RANSAC aligners
    are run before scoring against ground truth
  repo: https://github.com/mzmine/mzmine2
- name: RANSAC aligner
  role: Aligner under evaluation; iteratively fits alignment functions to matching
    peak pairs and discards non-matching outliers
- name: Join aligner
  role: Baseline aligner compared against RANSAC in the precision/recall table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:9a53568861300db92b70bb979ae9326889a3fd6f90f87c8c8df3991fab33935e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Join aligner
  - MZmine 2
  - RANSAC aligner
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precision-recall-computation

## Summary

Computes precision and recall of peak list alignments by scoring aligned peak lists against ground-truth alignment tables, as used in MZmine 2 to benchmark the RANSAC aligner against the Join aligner on the Lange et al. proteomic (P1, P2) and metabolomic (M1, M2) datasets.

## When to use

Use when evaluating peak list alignment quality against known ground-truth alignments, e.g. comparing aligners (RANSAC vs Join) across LC-MS fractions at several RT tolerances.

## When NOT to use

- The peak lists have no ground-truth alignment tables, so precision/recall cannot be scored against reference pairings
- The goal is de novo detection or gap filling of features rather than judging the correctness of an existing alignment
- Only one aligner is being run with no comparison baseline — precision/recall vs ground truth is chiefly a comparative benchmark here

## Inputs

- Real proteomic (P1, P2) and metabolomic (M1, M2) LC-MS peak lists from Lange et al.
- Tables of ground-truth alignments accompanying the Lange et al. datasets
- Aligned peak lists produced by the Join aligner and RANSAC aligner at RT tolerances 5 s, 20 s, 50 s

## Outputs

- Per-alignment precision (fraction of aligned peak pairs that are correct) and recall (fraction of true peak pairs aligned) values
- A comparison table of precision/recall across the 13 pairwise alignments for both aligners

## How to apply

For each pairwise alignment, import the peak lists into MZmine 2 while preserving the Lange et al. ground-truth alignment labels. Run both aligners at identical RT tolerances of 5 s, 20 s, and 50 s with m/z tolerance fixed, so the only varying factor is the aligner. Score each aligned peak list against the ground truth: precision is the fraction of aligned peak pairs that are correct, and recall is the fraction of true peak pairs that are aligned. Report values per dataset, fraction, aligner, and RT tolerance, then assemble them into a comparison table over the 13 pairwise alignments (P1 and P2 fractions, M1, M2). Validation: the expected pattern is that the RANSAC aligner wins in 11 of 13 alignments, is best on all P1 fractions, and loses only on P2 fraction 00 (which has over 6800 rows after alignment).

## Related tools

- **MZmine 2** (Platform in which peak lists are imported and the Join and RANSAC aligners are run before scoring against ground truth) — https://github.com/mzmine/mzmine2
- **RANSAC aligner** (Aligner under evaluation; iteratively fits alignment functions to matching peak pairs and discards non-matching outliers)
- **Join aligner** (Baseline aligner compared against RANSAC in the precision/recall table)

## Evaluation signals

- Reproduce the reported pattern: RANSAC better than Join in 11 of 13 alignments, with worse results only on P2 dataset fraction 00
- Verify RANSAC was best on all P1 fractions
- Precision and recall fall within [0, 1] for every dataset, fraction, aligner, and RT tolerance combination
- The comparison table covers all 13 pairwise alignments (P1 and P2 fractions, M1, M2) for both aligners and all three RT tolerances (5 s, 20 s, 50 s)

## Limitations

- On the P2 dataset fraction 00, the high number of features (over 6800 rows after alignment) made the Join aligner outperform RANSAC
- Scoring depends entirely on the correctness of the Lange et al. ground-truth alignment tables
- Results were validated mainly on LC-MS data; the MZmine 2 modules were tested mainly on LC-MS data

## Evidence

- [results] RANSAC vs Join benchmark result: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Ground-truth datasets used: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [supplementary] Synthetic benchmark scoring: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [results] Perfect synthetic performance: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] P2 fraction 00 row count: "the high number of features in this fraction (over 6800 rows after alignment)"
