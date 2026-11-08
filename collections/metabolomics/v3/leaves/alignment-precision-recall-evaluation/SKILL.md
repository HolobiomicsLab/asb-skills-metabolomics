---
name: alignment-precision-recall-evaluation
description: Use when you must quantitatively compare a peak list alignment method's
  output rows to known ground-truth correspondences, or reproduce the RANSAC vs Join
  aligner benchmark on synthetic lipidomic and real Lange et al. datasets.
when_to_use_negative:
- The data are raw LC-MS profiles with no detected peaks or peak lists — peak detection
  must be done first
- There is no ground-truth alignment table, so TP/FP/FN counts cannot be established
- You only need to run alignment for downstream analysis, not to benchmark alignment
  quality
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Platform whose RANSAC and Join peak list alignment modules are run on the
    benchmark peak lists
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK) version 12
  role: Required JDK version for building MZmine from source when implementing/reproducing
    the Join aligner baseline in Java
- name: Java
  role: Language used to implement the Join aligner baseline and its precision/recall
    evaluation
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:d0f75d0006737f8c1645a4ed6dda0241b747f4a11c7f3f17a51e210fecfae881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Java Development Kit (JDK) version 12
  - MZmine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# alignment-precision-recall-evaluation

## Summary

Benchmark a peak list alignment algorithm (e.g., the Join aligner or the RANSAC aligner in MZmine 2) by comparing its aligned rows against ground-truth alignment tables and computing precision (TP/(TP+FP)) and recall (TP/(TP+FN)) per dataset, as done for LC-MS/proteomic/metabolomic peak alignment in MZmine 2.

## When to use

Use when you must quantitatively compare a peak list alignment method's output rows to known ground-truth correspondences, or reproduce the RANSAC vs Join aligner benchmark on synthetic lipidomic and real Lange et al. datasets.

## When NOT to use

- The data are raw LC-MS profiles with no detected peaks or peak lists — peak detection must be done first
- There is no ground-truth alignment table, so TP/FP/FN counts cannot be established
- You only need to run alignment for downstream analysis, not to benchmark alignment quality

## Inputs

- 12 synthetic lipidomic peak lists (20 samples each, random non-linear RT deviation)
- Real proteomic peak lists P1 and P2 (Lange et al.) with ground-truth alignment tables
- Real metabolomic peak lists M1 and M2 (Lange et al.) with ground-truth alignment tables
- Ground-truth alignment correspondence tables
- User-defined m/z tolerance and retention-time tolerance parameters (e.g. 5 s, 20 s, 50 s RT)

## Outputs

- Per-dataset precision (TP/(TP+FP)) and recall (TP/(TP+FN)) values in a CSV (e.g. join_aligner_precision_recall.csv, ransac_precision_recall_results.csv)
- Aligned peak list rows per dataset
- TP/FP/FN counts per benchmark alignment

## How to apply

Load the benchmark peak lists — the 12 synthetic lipidomic datasets (each 20 samples seeded from one of 12 lipidomic studies with random non-linear retention time deviation) and the real proteomic (P1, P2) and metabolomic (M1, M2) datasets from Lange et al. with their ground-truth alignment tables — into MZmine 2. Run the alignment module under test, e.g. the Join aligner, which for each detected peak finds matches within user-defined m/z tolerance and a fixed retention-time tolerance and groups them into alignment rows, creating a new row when no candidate fits. For RT tolerance sweeps use 5 s, 20 s, and 50 s as in the RANSAC evaluation. Compare each aligned peak list row-by-row against the ground-truth table to count true positives, false positives, and false negatives, then compute precision = TP/(TP+FP) and recall = TP/(TP+FN) per dataset (values reported on a 0–100 scale). Write per-dataset values to a CSV such as join_aligner_precision_recall.csv or ransac_precision_recall_results.csv, mirroring the supplementary comparison of RANSAC vs Join aligner results.

## Related tools

- **MZmine 2** (Platform whose RANSAC and Join peak list alignment modules are run on the benchmark peak lists) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK) version 12** (Required JDK version for building MZmine from source when implementing/reproducing the Join aligner baseline in Java)
- **Java** (Language used to implement the Join aligner baseline and its precision/recall evaluation) — https://github.com/mzmine/mzmine2

## Evaluation signals

- Output CSV exists with precision and recall columns, every value in the range 0–100, and one entry per benchmark alignment
- On the 12 synthetic lipidomic datasets at RT tolerances 5 s, 20 s, and 50 s, the RANSAC aligner reproduces 100% precision and recall
- The Join aligner baseline is worse than RANSAC in 11 of 13 alignments, with worse results only in the P2 dataset fraction 00 case (over 6800 rows after alignment)
- TP/FP/FN counts re-derived from row-wise comparison against the ground-truth tables reproduce the reported precision/recall values
- Aligned row counts match the source peak list dimensions (e.g. >6800 rows for P2 fraction 00)

## Limitations

- The Join aligner cannot cope with non-linear deviation of retention times among samples, degrading its precision/recall on the synthetic datasets with random non-linear RT deviation
- Results depend on the chosen m/z and retention-time tolerances (5 s, 20 s, 50 s sweeps); different tolerances change the outcome
- The evaluation relies on ground-truth tables that exist only for the curated synthetic datasets and the Lange et al. P1/P2/M1/M2 datasets
- Developed modules were tested mainly on LC-MS data only, so generalization to other data types is not demonstrated

## Evidence

- [results] RANSAC 100% on synthetic datasets: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] RANSAC vs Join aligner 11 of 13: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Synthetic dataset construction: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [results] Ground-truth benchmark datasets: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [supplementary] Supplementary precision/recall comparison: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [readme] JDK requirement from README: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
