---
name: ransac-robust-regression-alignment
description: Use when aligning LC-MS peak lists across sample batches with random
  non-linear RT deviations, and you must not tolerate spurious row correspondences
  (precision/recall critical).
when_to_use_negative:
- When the dataset is a real proteomic fraction with extremely high feature density,
  e.g., P2 fraction 00 with over 6800 rows after alignment, where the Join aligner
  outperformed RANSAC in one reported case
- When input is already a single aligned feature table and no cross-sample RT correction
  is needed
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine 2
  role: Host framework providing the RANSAC peak list alignment module and the Join
    aligner baseline
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Platform-independent implementation language for building and extending MZmine
    2
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:ced7c964ee3608537371b7361ba9c596f25c3d27e6b6de418350ac84c347ed52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ransac-robust-regression-alignment

## Summary

Peak list alignment in MZmine 2 using a random sample consensus (RANSAC)-based method that robustly models non-linear retention time deviations between samples, achieving 100% precision and recall on synthetic lipidomic datasets where the Join aligner failed.

## When to use

Use when aligning LC-MS peak lists across sample batches with random non-linear RT deviations, and you must not tolerate spurious row correspondences (precision/recall critical).

## When NOT to use

- When the dataset is a real proteomic fraction with extremely high feature density, e.g., P2 fraction 00 with over 6800 rows after alignment, where the Join aligner outperformed RANSAC in one reported case
- When input is already a single aligned feature table and no cross-sample RT correction is needed

## Inputs

- MZmine 2 peak lists from 12 synthetic lipidomic datasets (seed sample from each of 12 lipidomic studies expanded to 20 samples with random non-linear RT deviation)
- Ground-truth correspondence tables for the aligned peaks
- Benchmark peak lists from Lange et al. proteomic (P1, P2) and metabolomic (M1, M2) datasets with ground-truth alignments

## Outputs

- Aligned peak list per dataset and RT tolerance (5 s, 20 s, 50 s)
- Precision and recall values per dataset and tolerance (e.g., ransac_precision_recall_results.csv)

## How to apply

Import the raw or derived peak lists into MZmine 2 and run the RANSAC peak list alignment module instead of the Join aligner. Configure the RT tolerance sweep (e.g., 5 s, 20 s, 50 s) and let RANSAC iteratively fit a retention-time model while discarding outlier correspondences, which handles the random non-linear RT deviation present in the 12 synthetic lipidomic datasets (20 samples each, seeded from real lipidomic studies). Validate each aligned peak list against known ground-truth correspondences (e.g., the synthetic ground truth, or the Lange et al. P1/P2/M1/M2 tables) and compute precision and recall per dataset and tolerance. Aggregate results; the expected reference outcome is that only RANSAC reaches 100% in both precision and recall on the synthetic sets, while the Join aligner does not. On real benchmark data (13 alignments from P1, P2, M1, M2), expect RANSAC to be better in 11 of 13 cases.

## Related tools

- **MZmine 2** (Host framework providing the RANSAC peak list alignment module and the Join aligner baseline) — https://github.com/mzmine/mzmine2
- **Java** (Platform-independent implementation language for building and extending MZmine 2) — https://github.com/mzmine/mzmine2

## Examples

```
/opt/mzmine/bin/mzmine -batch ransac_alignment.xml
```

## Evaluation signals

- RANSAC alignment must yield precision = 100% and recall = 100% on all 12 synthetic datasets at the tested RT tolerances
- Join aligner results must NOT reach perfect precision/recall on the same synthetic datasets (control comparison)
- On Lange et al. benchmark alignments, RANSAC should beat the other aligner in 11 of 13 alignments, with a single loss on P2 dataset fraction 00
- Every dataset/tolerance combination must produce a precision and recall value in the output (12 datasets x 3 tolerances)
- Aligned peak list row counts should be consistent with ground-truth correspondences rather than inflated gap rows

## Limitations

- RANSAC alignment was worse than Join aligner in a single case, the P2 dataset fraction 00 with over 6800 rows after alignment
- MZmine 2 modules were tested mainly on LC-MS data; validation on other data types (e.g., GC-MS) is not demonstrated
- The synthetic evaluation is limited to 12 lipidomic-derived datasets with random non-linear RT deviation

## Evidence

- [results] RANSAC achieves perfect alignment on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [abstract] Method is a new peak list alignment approach: "a new method for peak list alignment based on the random sample consensus (RANSAC) algorithm"
- [results] Synthetic dataset construction: "12 synthetic datasets were created using samples from 12 different lipidomic studies."
- [results] Benchmark vs Join aligner on real data: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [readme] Software context: "MZmine 2 is an open-source software for mass-spectrometry data processing."
