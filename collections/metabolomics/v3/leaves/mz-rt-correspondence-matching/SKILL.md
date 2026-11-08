---
name: mz-rt-correspondence-matching
description: Use when multiple per-sample peak lists from LC-MS runs must be merged
  into a single master peak list by matching peaks within user-defined m/z and retention-time
  tolerances, e.g. for benchmarking alignment methods.
when_to_use_negative:
- Retention times deviate non-linearly among samples — the Join aligner cannot cope
  with non-linear RT deviation; a RANSAC-based aligner is preferable
- A feature table already aligned across samples exists, so re-matching is redundant
- Evaluating against ground truth is impossible because no reference alignment table
  is available
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Framework providing the Join aligner module for peak list matching; the alignment
    baseline is implemented/run within it
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Implementation language used to code the Join aligner matching logic
- name: Java Development Kit (JDK) version 12
  role: Build requirement for compiling/running the MZmine 2 code containing the aligner
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:5a19f67bd9b1139c9488054743aadc2464270d45eee71fe8e782624283544a06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Java Development Kit (JDK) version 12
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mz-rt-correspondence-matching

## Summary

Matches detected peaks across multiple LC-MS peak lists into alignment rows using m/z and retention-time tolerance windows, as implemented by the MZmine Join aligner, the original MZmine peak list alignment algorithm.

## When to use

Use when multiple per-sample peak lists from LC-MS runs must be merged into a single master peak list by matching peaks within user-defined m/z and retention-time tolerances, e.g. for benchmarking alignment methods.

## When NOT to use

- Retention times deviate non-linearly among samples — the Join aligner cannot cope with non-linear RT deviation; a RANSAC-based aligner is preferable
- A feature table already aligned across samples exists, so re-matching is redundant
- Evaluating against ground truth is impossible because no reference alignment table is available

## Inputs

- Per-sample peak lists from LC-MS runs (e.g. 12 synthetic lipidomic datasets with 20 samples each)
- Real proteomic and metabolomic peak lists (P1, P2, M1, M2 from Lange et al.)
- User-specified m/z tolerance and fixed retention-time tolerance for the alignment window
- Ground-truth alignment tables for evaluation

## Outputs

- Aligned master peak list with one alignment row per matched feature group
- Per-dataset precision (TP/(TP+FP)) and recall (TP/(TP+FN)) values, e.g. written to join_aligner_precision_recall.csv

## How to apply

Start from an empty master peak list, then for each sample peak find the best candidate in the master list within a two-dimensional alignment window defined by user-specified m/z and retention-time tolerances; if no suitable candidate is found, create a new row in the master list, otherwise group the match into the existing alignment row. Implement this in Java (JDK 12 or newer) against the MZmine 2 codebase, or reuse the Join aligner module directly. Run the aligner on each input peak list set — e.g. the 12 synthetic lipidomic datasets (20 samples each) and the Lange et al. proteomic (P1, P2) and metabolomic (M1, M2) datasets — to produce aligned peak lists. Validate by counting true positives, false positives, and false negatives against the ground-truth alignment tables and computing precision (TP/(TP+FP)) and recall (TP/(TP+FN)) per dataset.

## Related tools

- **MZmine 2** (Framework providing the Join aligner module for peak list matching; the alignment baseline is implemented/run within it) — https://github.com/mzmine/mzmine2
- **Java** (Implementation language used to code the Join aligner matching logic)
- **Java Development Kit (JDK) version 12** (Build requirement for compiling/running the MZmine 2 code containing the aligner)

## Examples

```
./gradlew && ./startMZmine --batch join_aligner_batch.xml --mzTolerance 0.001 --rtTolerance 0.5
```

## Evaluation signals

- Output file join_aligner_precision_recall.csv exists with precision and recall columns, one entry per benchmark alignment
- All precision and recall values fall in the range 0–100
- Join aligner reproduces published baseline behavior: worse than RANSAC in 11 of 13 alignments
- The single worse-than-RANSAC exception is the P2 dataset fraction 00 case, exceeding 6800 rows after alignment
- Every detected peak is either matched to an existing master-list row (within m/z and RT tolerances) or placed in a newly created row — no peaks are dropped silently

## Limitations

- The Join aligner is unable to cope with a non-linear deviation of the retention times among samples
- Alignment quality depends on user-specified m/z and retention-time tolerance choices; no automatic tolerance estimation is described
- On datasets with a very high number of features (e.g. P2 fraction 00, over 6800 rows after alignment) the baseline may produce degenerate alignment results

## Evidence

- [results] Join aligner two-dimensional alignment window: [evidence span withheld — anchor status: not_found]
- [results] Join aligner limitation: "One disadvantage of the Join aligner is the inability to cope with a non-linear deviation of the retention times among samples."
- [results] RANSAC vs Join aligner benchmark outcome: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Benchmark datasets: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [readme] JDK build requirement: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
- [supplementary] Supplementary precision/recall comparison: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
