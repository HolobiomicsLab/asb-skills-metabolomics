---
name: peak-list-alignment
description: Use when combining detected peak lists from multiple LC-MS samples into
  one master peak list in MZmine 2 and retention times drift non-linearly between
  samples, forcing a choice between the Join and RANSAC aligners.
when_to_use_negative:
- Input is raw spectra rather than peak lists — alignment modules consume peak lists
  produced after peak picking and peak list deisotoping
- Only a single sample's peak list exists, so there is nothing to align across samples
- Non-LC-MS data types (e.g., GC-MS) — the paper notes the developed modules were
  tested mainly on LC-MS data
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Modular framework whose alignment modules consume imported peak lists and
    produce the aligned master peak list; built from source with ./gradlew
  repo: https://github.com/mzmine/mzmine2
- name: Join aligner
  role: Baseline aligner matching each peak to the best master-list row within a fixed
    2D m/z/RT window; also the final matching step executed inside AWi of the RANSAC
    aligner
  repo: https://github.com/mzmine/mzmine2
- name: RANSAC aligner
  role: Aligner that iteratively samples peak correspondences, fits a retention-time
    correspondence model with RANSAC, LOESS-smooths it, recenters AWi, and discards
    outlier matches
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK)
  role: Toolchain (version 12 or newer for mzmine2) required to compile the framework
    before implementing or running the alignment modules
  repo: http://jdk.java.net
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:7015c3e4abd68f427bbf4ce7e6a231fd4bbd47abd16cd6b434fce8f148791ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java Development Kit (JDK)
  - Join aligner
  - MZmine 2
  - RANSAC aligner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# peak-list-alignment

## Summary

Aligns per-sample LC-MS peak lists into a single master peak list in MZmine 2, either with the Join aligner's fixed 2D m/z–RT window or with the RANSAC-based aligner that fits a LOESS-smoothed retention-time correspondence model to handle non-linear RT shifts; performance is judged by precision/recall against ground-truth alignments.

## When to use

Use when combining detected peak lists from multiple LC-MS samples into one master peak list in MZmine 2 and retention times drift non-linearly between samples, forcing a choice between the Join and RANSAC aligners.

## When NOT to use

- Input is raw spectra rather than peak lists — alignment modules consume peak lists produced after peak picking and peak list deisotoping
- Only a single sample's peak list exists, so there is nothing to align across samples
- Non-LC-MS data types (e.g., GC-MS) — the paper notes the developed modules were tested mainly on LC-MS data

## Inputs

- Per-sample LC-MS peak lists already peak-picked and deisotoped, loaded into MZmine 2
- User-set parameters: m/z and RT tolerances for the Join aligner (e.g., RT 5/20/50 s); RW thresholds rm0 and rr0 and AW RT threshold ar0 for the RANSAC aligner
- Ground-truth alignment tables (Lange et al. P1, P2, M1, M2) when benchmarking precision and recall

## Outputs

- Aligned master peak list with one row per peak matched across samples
- Per-dataset precision and recall tables, e.g., ransac_alignment_precision_recall.csv and join_aligner_precision_recall.csv

## How to apply

Import peak-picked, deisotoped peak lists into MZmine 2 (alignment sits between peak list deisotoping and gap filling/normalization) and choose the Join aligner for near-linear RT behavior or the RANSAC aligner when RT shifts are non-linear. With the Join aligner, build an empty master peak list and match each sample peak to the best master row inside a user-set 2D window of m/z and RT tolerance (the Lange et al. benchmark swept RT tolerances of 5, 20, and 50 s at fixed m/z tolerance); unmatched peaks open a new row. With the RANSAC aligner, define two 2D windows — RW (m/z threshold rm0, RT threshold rr0) and AW (same rm0, RT threshold ar0) — then for each master-list row mark candidate alignments in RWi, iteratively sample random peak correspondences between the reference peak list and sample Sj, fit a retention-time correspondence model, keep inliers within RW, smooth the model with LOESS regression, predict the RT correction to recenter AWi, and finally run the Join algorithm within AWi. Score each aligned peak list against ground-truth alignment tables (Lange et al. P1, P2, M1, M2) using precision = TP/(TP+FP) and recall = TP/(TP+FN), repeating for both aligners.

## Related tools

- **MZmine 2** (Modular framework whose alignment modules consume imported peak lists and produce the aligned master peak list; built from source with ./gradlew) — https://github.com/mzmine/mzmine2
- **Join aligner** (Baseline aligner matching each peak to the best master-list row within a fixed 2D m/z/RT window; also the final matching step executed inside AWi of the RANSAC aligner) — https://github.com/mzmine/mzmine2
- **RANSAC aligner** (Aligner that iteratively samples peak correspondences, fits a retention-time correspondence model with RANSAC, LOESS-smooths it, recenters AWi, and discards outlier matches) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK)** (Toolchain (version 12 or newer for mzmine2) required to compile the framework before implementing or running the alignment modules) — http://jdk.java.net

## Examples

```
git clone https://github.com/mzmine/mzmine2 && cd mzmine2 && ./gradlew
```

## Evaluation signals

- Per-dataset precision TP/(TP+FP) and recall TP/(TP+FN) values fall in the 0–100 range with one entry per benchmark alignment in the output CSVs
- On the 12 synthetic lipidomic datasets (20 samples each, seeded from one sample per study), the RANSAC aligner reproduces the published 100% precision and 100% recall
- RANSAC outperforms Join in 11 of 13 real-data alignments, is best on all P1 fractions, and is worse in exactly one case (P2 dataset fraction 00)
- Both ransac_alignment_precision_recall.csv and join_aligner_precision_recall.csv exist with precision and recall columns covering the 12 synthetic datasets
- The aligned master peak list contains no duplicate rows for the same peak and each row holds one peak per sample within the chosen m/z tolerance

## Limitations

- The Join aligner cannot cope with non-linear deviation of the retention times among samples, so it underperforms RANSAC under RT drift
- Known failure case: P2 dataset fraction 00, where the high number of features (over 6800 rows after alignment) makes the RANSAC aligner worse than the Join aligner
- MZmine 2 modules were tested mainly on LC-MS data; GC-MS support and MASCOT interoperation were described only as future expansion
- The mzmine2 repository is kept as reference only — development has moved to mzmine/mzmine3, which requires a newer JDK (23+ per the current README) than the paper-era mzmine2 (JDK 12+)

## Evidence

- [abstract] RANSAC alignment method introduction: "a new method for peak list alignment based on the random sample consensus (RANSAC) algorithm"
- [results] RANSAC 100% precision and recall on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] RANSAC vs Join on real datasets: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [supplementary] Supplementary precision/recall benchmark scope: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [readme] JDK requirement for building mzmine2: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
