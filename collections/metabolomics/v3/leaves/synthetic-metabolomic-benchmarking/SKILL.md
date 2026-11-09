---
name: synthetic-metabolomic-benchmarking
description: Use when comparing alignment algorithms on peak lists where no experimental
  ground truth exists, by creating synthetic replicates with random non-linear RT
  deviation so true peak correspondences are known a priori.
when_to_use_negative:
- The input is already real experimental data with no ground-truth alignment table
  — the synthetic seed-expansion scheme requires known true correspondences by construction
- The goal is general MS data processing (peak picking, deisotoping, gap filling)
  rather than benchmarking of alignment algorithms specifically
- Only a single aligner is of interest with no comparison target — precision/recall
  benchmarking is only meaningful relative to the ground truth and, ideally, a competing
  aligner such as the Join aligner
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine 2
  role: Platform in which the synthetic peak lists are imported and the RANSAC and
    Join aligner modules are configured and run at RT tolerances of 5, 20, and 50
    s
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Language MZmine 2 is implemented in; building MZmine 2 from source (./gradlew)
    requires the Java Development Kit, needed if replicating the aligner modules locally
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:932d551dbb5f27ebab85358f0448908cb50d35eb3d03f584baeb7e26f9955c17
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
yes_rate: 1.0
n_claims: 10
---

# synthetic-metabolomic-benchmarking

## Summary

Benchmark peak list alignment algorithms by generating synthetic metabolomic/lipidomic datasets from a seed sample with known ground-truth correspondences, then scoring precision and recall of each aligner (e.g. RANSAC vs Join aligner in MZmine 2).

## When to use

Use when comparing alignment algorithms on peak lists where no experimental ground truth exists, by creating synthetic replicates with random non-linear RT deviation so true peak correspondences are known a priori.

## When NOT to use

- The input is already real experimental data with no ground-truth alignment table — the synthetic seed-expansion scheme requires known true correspondences by construction
- The goal is general MS data processing (peak picking, deisotoping, gap filling) rather than benchmarking of alignment algorithms specifically
- Only a single aligner is of interest with no comparison target — precision/recall benchmarking is only meaningful relative to the ground truth and, ideally, a competing aligner such as the Join aligner

## Inputs

- 12 synthetic lipidomic peak lists (each derived from a seed sample of one of 12 lipidomic studies, expanded to 20 samples with random non-linear RT deviation)
- Known ground-truth alignment correspondences for each synthetic dataset
- MZmine 2 project with the synthetic peak lists imported

## Outputs

- ransac_precision_recall_results.csv with precision and recall per dataset per RT tolerance (5 s, 20 s, 50 s)
- Per-aligner precision/recall comparison (RANSAC vs Join aligner) across the 12 synthetic datasets

## How to apply

Take a single seed sample from each source study and expand it into a synthetic set of 20 samples by applying random non-linear retention time deviations, so that the true peak correspondences across samples are known by construction. Import the resulting 12 synthetic lipidomic peak lists into MZmine 2 and run the RANSAC peak list alignment module at a series of retention-time tolerances (5 s, 20 s, and 50 s). Run the competing aligner — the Join aligner — on the same datasets at the same tolerances. Compare each aligned peak list against the known synthetic ground-truth correspondences to compute precision and recall per dataset and per tolerance, aggregating all 12 datasets x 3 tolerances into a results CSV. The benchmark has a known expected outcome: only the RANSAC algorithm achieved 100% in both precision and recall on these synthetic data sets.

## Related tools

- **MZmine 2** (Platform in which the synthetic peak lists are imported and the RANSAC and Join aligner modules are configured and run at RT tolerances of 5, 20, and 50 s) — https://github.com/mzmine/mzmine2
- **Java** (Language MZmine 2 is implemented in; building MZmine 2 from source (./gradlew) requires the Java Development Kit, needed if replicating the aligner modules locally) — https://github.com/mzmine/mzmine2

## Evaluation signals

- Validation: the output must reproduce the reported result that the RANSAC algorithm achieved 100% in both precision and recall on the synthetic data sets
- The Join aligner must fail to reach perfect (100%) precision and recall at the tested retention time tolerances (5 s, 20 s, 50 s), matching the reported negative result
- Each synthetic dataset contains exactly 20 samples derived from a single seed sample, so row counts and correspondence counts must be consistent with that construction
- Precision and recall values must be computed against the known synthetic ground-truth correspondences, not against another aligner's output
- Results must be aggregated over all 12 datasets x 3 tolerances (36 cells) in ransac_precision_recall_results.csv

## Limitations

- The synthetic benchmark covers lipidomic data with random non-linear RT deviation; complementary evaluation on real experimental data used only the four proteomic/metabolomic datasets (P1, P2, M1, M2) from Lange et al. with their ground-truth alignment tables
- On real experimental data the RANSAC aligner is not universally superior: it provided better results in 11 of 13 alignments, with worse results in a single case (P2 dataset fraction 00), where the high number of features (over 6800 rows after alignment) was cited
- The ground-truth data and Tranche hash are referenced but the hash value is absent from the manuscript text, complicating retrieval of the original datasets
- This is a benchmarking-of-aligners skill, not a validated processing pipeline; the paper's modules were tested mainly on LC-MS data

## Evidence

- [results] Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets.: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] 12 synthetic datasets from lipidomic studies: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [supplementary] supplementary precision/recall table description: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [results] real-data benchmark datasets from Lange et al.: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [results] RANSAC vs Join aligner comparison on experimental data: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
