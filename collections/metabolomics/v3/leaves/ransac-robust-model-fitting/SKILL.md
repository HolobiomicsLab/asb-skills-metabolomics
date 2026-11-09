---
name: ransac-robust-model-fitting
description: Use when aligning LC-MS peak lists across samples where spurious peak
  correspondences (outliers) would corrupt a single least-squares fit, and a robust
  random-sampling model fit with m/z and RT tolerance windows is needed.
when_to_use_negative:
- The input is raw mass-spectrometry data requiring data import, filtering/smoothing,
  peak picking, or deisotoping first — RANSAC alignment operates on already-generated
  peak lists.
- A simple Join aligner at fixed RT tolerance (e.g. 5 s, 20 s, 50 s) suffices and
  robust outlier rejection is not needed — though note the Join aligner was outperformed
  by RANSAC in 11 of 13 real-data alignments.
- The data type is not LC-MS; the paper's developed modules were tested mainly on
  LC-MS data only.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Modular open-source framework whose peak list alignment module implements
    the RANSAC-based alignment with RW/AW parameters
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK) version 12
  role: Required to compile and extend the MZmine 2 framework (built with ./gradlew)
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:4b2fa02fadd07f7e4f062feb0d7223f0cde56e182b2aa3d11791af2a84f7033e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java Development Kit (JDK) version 12
  - Join aligner
  - MZmine 2
  - RANSAC aligner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ransac-robust-model-fitting

## Summary

Robust iterative model fitting with the Random Sample Consensus (RANSAC) algorithm, as implemented in MZmine 2 for peak list alignment: fit a retention-time correspondence model to random peak-pair samples, retain inliers within tolerance windows, and use the model (with LOESS regression) to correct retention times and constrain peak matching.

## When to use

Use when aligning LC-MS peak lists across samples where spurious peak correspondences (outliers) would corrupt a single least-squares fit, and a robust random-sampling model fit with m/z and RT tolerance windows is needed.

## When NOT to use

- The input is raw mass-spectrometry data requiring data import, filtering/smoothing, peak picking, or deisotoping first — RANSAC alignment operates on already-generated peak lists.
- A simple Join aligner at fixed RT tolerance (e.g. 5 s, 20 s, 50 s) suffices and robust outlier rejection is not needed — though note the Join aligner was outperformed by RANSAC in 11 of 13 real-data alignments.
- The data type is not LC-MS; the paper's developed modules were tested mainly on LC-MS data only.

## Inputs

- Peak lists from LC-MS samples (e.g. after peak picking and deisotoping in MZmine 2)
- Ground-truth alignment tables (e.g. from the Lange et al. P1, P2, M1, M2 datasets) for scoring
- User-set parameters: RW m/z threshold rm0, RW RT threshold rr0, AW RT threshold ar0

## Outputs

- Aligned peak list (master list) across all samples
- Per-alignment precision and recall values (e.g. written to ransac_alignment_precision_recall.csv)

## How to apply

For each sample peak list Sj to be aligned to the master list L, define two user-set two-dimensional windows: the RANSAC window (RW, with m/z threshold rm0 and RT threshold rr0) and the alignment window (AW, with the same rm0 and RT threshold ar0). Repeatedly sample random peak correspondences between the reference and target lists, fit a retention-time correspondence model to each sample, and score inlier peaks lying within the RW tolerance; keep the model with the most inliers. Apply LOESS regression to the winning model to predict the retention-time correction, recenter AWi accordingly, and then apply the Join algorithm within AWi to match remaining peaks. Iterate over all sample peak lists to produce the aligned peak list, which is then scored for precision (fraction of aligned peak pairs that are correct) and recall (fraction of true peak pairs that are aligned) against ground-truth alignment tables.

## Related tools

- **MZmine 2** (Modular open-source framework whose peak list alignment module implements the RANSAC-based alignment with RW/AW parameters) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK) version 12** (Required to compile and extend the MZmine 2 framework (built with ./gradlew)) — https://github.com/mzmine/mzmine2

## Examples

```
git clone https://github.com/mzmine/mzmine2 && cd mzmine2 && ./gradlew
```

## Evaluation signals

- On synthetic lipidomic datasets (12 sets, 20 samples each from one seed sample per study), the RANSAC algorithm must achieve 100% in both precision and recall.
- On the Lange et al. real datasets (P1, P2, M1, M2), RANSAC should give better results than the Join aligner in 11 of 13 alignments and be best on all P1 fractions.
- The only expected worse case should be the P2 dataset fraction 00, which has over 6800 rows after alignment.
- The fitted retention-time correspondence model, after LOESS regression, must correctly recenter the alignment window AWi before peak matching.
- Comparison runs with the Join aligner at identical RT tolerances (5 s, 20 s, 50 s, fixed m/z tolerance) should be produced for a fair precision/recall table.

## Limitations

- Performance was worse than the Join aligner in one of 13 real-data alignments (P2 dataset fraction 00), likely related to the high number of features in that fraction (over 6800 rows after alignment).
- MZmine 2 modules were tested mainly on LC-MS data; validation on other data types (e.g. GC-MS) is not demonstrated.
- The method requires user-set window parameters (rm0, rr0 for RW; ar0 for AW), which must be chosen appropriately for the dataset.

## Evidence

- [results] The RANSAC alignment defines two user-set two-dimensional windows (RW with m/z threshold rm0 and retention time threshold rr0; AW with the same rm0 but threshold ar0): [evidence span withheld — anchor status: not_found]
- [results] Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets.: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00): "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [readme] MZmine development requires Java Development Kit (JDK) version 12 or newer: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
