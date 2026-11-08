---
name: lc-ms-retention-time-drift-correction
description: Use when aligning peak lists from multiple LC-MS samples whose retention
  times drift nonlinearly, and a naive join-by-m/z/RT tolerance fails; requires per-sample
  peak lists plus m/z and RT thresholds.
when_to_use_negative:
- Input is raw LC-MS data that has not yet been peak-picked — the method operates
  on peak lists, not raw data files
- The dataset fraction contains a very high number of features (e.g., over 6800 rows
  after alignment, as in the P2 dataset fraction 00) where the Join aligner outperformed
  RANSAC
- Processing non-LC-MS data types — the developed MZmine 2 modules were tested mainly
  on LC-MS data only
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine 2
  role: Modular framework in which the RANSAC peak list alignment module is implemented
    and run
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK)
  role: Toolchain required to compile the MZmine 2 framework (JDK version 12 or newer)
    and develop the alignment module
- name: Join aligner
  role: Baseline peak list alignment algorithm used for precision/recall comparison
    against the RANSAC aligner
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:21857efe0318f7184c427c5410b54b1aab0d449e99e4691e413e3e3c3da91ce2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java Development Kit (JDK)
  - Join aligner
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# LC-MS retention time drift correction

## Summary

Corrects retention-time drift across LC-MS peak lists by fitting a RANSAC-based correspondence model (plus LOESS regression) between each sample and a master peak list, as implemented in the MZmine 2 RANSAC aligner, and matches remaining peaks within an alignment window.

## When to use

Use when aligning peak lists from multiple LC-MS samples whose retention times drift nonlinearly, and a naive join-by-m/z/RT tolerance fails; requires per-sample peak lists plus m/z and RT thresholds.

## When NOT to use

- Input is raw LC-MS data that has not yet been peak-picked — the method operates on peak lists, not raw data files
- The dataset fraction contains a very high number of features (e.g., over 6800 rows after alignment, as in the P2 dataset fraction 00) where the Join aligner outperformed RANSAC
- Processing non-LC-MS data types — the developed MZmine 2 modules were tested mainly on LC-MS data only

## Inputs

- Per-sample LC-MS peak lists (m/z and retention time for each detected peak)
- A reference/master peak list L used as the alignment target
- Ground-truth alignment tables (e.g., from the Lange et al. P1, P2, M1, M2 datasets) for validation
- AW and RW parameter settings: rm0 (m/z threshold), rr0 (RT threshold for RW), ar0 (RT threshold for AW)

## Outputs

- A single aligned peak list spanning all samples
- Per-alignment precision and recall values (e.g., written to ransac_alignment_precision_recall.csv)
- The fitted retention-time correction model (RANSAC model with LOESS regression)

## How to apply

Implement or invoke the RANSAC peak list alignment module in MZmine 2 (Java, built from github.com/mzmine/mzmine2 with ./gradlew). For each sample peak list Sj aligned to master list L, define two user-set 2D windows: the RANSAC window (RW, with m/z threshold rm0 and retention-time threshold rr0) used to score inlier peak correspondences, and the alignment window (AW, same rm0 with threshold ar0) used for final matching. Repeatedly sample random peak correspondences between reference and target lists, fit a retention-time correspondence model, score inliers within the RW tolerance, apply LOESS regression to the fitted model, and use the predicted retention-time correction to recenter AWi before running the Join algorithm within AWi; iterate over all sample peak lists to produce the aligned peak list. Validate by computing precision and recall of the aligned peak list against ground-truth alignment tables, benchmarking against the Join aligner.

## Related tools

- **MZmine 2** (Modular framework in which the RANSAC peak list alignment module is implemented and run) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK)** (Toolchain required to compile the MZmine 2 framework (JDK version 12 or newer) and develop the alignment module)
- **Join aligner** (Baseline peak list alignment algorithm used for precision/recall comparison against the RANSAC aligner)

## Examples

```
git clone https://github.com/mzmine/mzmine2 && cd mzmine2 && ./gradlew
```

## Evaluation signals

- RANSAC aligner must achieve 100% precision and 100% recall on the 12 synthetic lipidomic datasets (20 samples each, generated from one seed sample per study), as reported in the paper
- On real data (P1, P2, M1, M2 from Lange et al.), the RANSAC aligner should outperform the Join aligner in the majority of the 13 alignments (11 of 13 in the paper), with ground-truth alignment tables as reference
- The predicted retention-time correction from the LOESS-regressed RANSAC model must recenter the alignment window AWi before the Join algorithm runs within it
- Aligned peak lists are scored against ground-truth tables for both RANSAC and Join aligners, and per-dataset values are written to ransac_alignment_precision_recall.csv

## Limitations

- The Join aligner performed better in a single real-data case (P2 dataset fraction 00, which had over 6800 rows after alignment), so RANSAC is not universally superior on large, dense peak lists
- Developed MZmine 2 modules were tested mainly on LC-MS data; validation on other data types (e.g., GC-MS) was not demonstrated
- The mzmine2 repository is archived — all development has moved to the mzmine/mzmine3 repository (which requires JDK 23+), so the original RANSAC alignment code lives in a legacy codebase

## Evidence

- [other] RANSAC alignment defines RW/AW windows: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim of RANSAC alignment method: "a new method for peak list alignment based on the random sample consensus (RANSAC) algorithm"
- [results] 100% precision/recall on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] Real-data comparison to Join aligner: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Synthetic benchmark dataset construction: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [readme] JDK requirement for building MZmine 2: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
