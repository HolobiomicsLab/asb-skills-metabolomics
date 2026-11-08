---
name: synthetic-benchmark-precision-recall-evaluation
description: Use when validating a peak-list alignment algorithm where the true row-to-feature
  correspondence across samples can be constructed, and precision/recall of aligned
  rows must be quantified on synthetic data plus real reference alignments.
when_to_use_negative:
- The analysis goal is peak detection, deisotoping, or gap filling rather than scoring
  the correspondence of aligned rows across samples against a known truth.
- No ground-truth alignment exists and no synthetic samples with known parent features
  can be generated, so precision and recall cannot be computed.
- Data are not peak lists from MS profiling experiments (e.g. GC-MS or proteomics
  search results) that the article's alignment evaluation was not demonstrated on.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Open-source LC-MS processing framework whose peak-list alignment modules (RANSAC
    aligner and Join aligner) are the objects being scored by the precision/recall
    benchmark; built with ./gradlew using JDK 12 or newer.
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK)
  role: Required toolchain for compiling the MZmine 2 source tree so the alignment
    modules under evaluation can be built and run.
- name: Join aligner
  role: Baseline alignment algorithm against which the RANSAC aligner's precision
    and recall are compared on synthetic and real datasets.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:9112e40b22a4208d1adaceeaf3a9e52d713e0a16b253b412ce9056b3e429ebe9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java Development Kit (JDK)
  - Join aligner
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Synthetic benchmark precision-recall evaluation

## Summary

Evaluation skill for assessing peak-list alignment accuracy by generating synthetic LC-MS replicate datasets from a seed sample with a known ground-truth alignment, then computing precision and recall of aligner output against that truth, as done for the RANSAC aligner in MZmine 2.

## When to use

Use when validating a peak-list alignment algorithm where the true row-to-feature correspondence across samples can be constructed, and precision/recall of aligned rows must be quantified on synthetic data plus real reference alignments.

## When NOT to use

- The analysis goal is peak detection, deisotoping, or gap filling rather than scoring the correspondence of aligned rows across samples against a known truth.
- No ground-truth alignment exists and no synthetic samples with known parent features can be generated, so precision and recall cannot be computed.
- Data are not peak lists from MS profiling experiments (e.g. GC-MS or proteomics search results) that the article's alignment evaluation was not demonstrated on.

## Inputs

- Seed LC-MS sample from each lipidomic study (12 studies) from which 20 synthetic samples per study are generated
- Real proteomic datasets P1 and P2 and metabolomic datasets M1 and M2 with tables of 'ground truth' alignments from Lange et al.
- Aligned peak lists produced by the aligner under test (e.g. RANSAC aligner) and by a comparison baseline (Join aligner)

## Outputs

- Per-dataset precision and recall values written to ransac_alignment_precision_recall.csv
- Quantified comparison of aligners (e.g. RANSAC vs Join aligner) across 12 synthetic and 13 real-data alignments

## How to apply

Build synthetic benchmark sets by taking one seed sample from each study (e.g. 12 different lipidomic studies) and generating 20 synthetic samples per study so the exact correspondence of each peak to its parent feature is known. Run the alignment module under test (e.g. the RANSAC aligner with its RW and AW windows, or the Join aligner as a baseline) on each synthetic dataset and on real reference datasets (P1, P2, M1, M2 from Lange et al.) that ship with published ground-truth alignment tables. Count an aligned row as a true positive when it contains the correct peaks from the ground truth, then compute precision = TP/(TP+FP) and recall = TP/(TP+FN) per dataset; write the per-dataset values to a CSV such as ransac_alignment_precision_recall.csv. Judge success against the paper's reported ceiling — the RANSAC algorithm achieved 100% precision and 100% recall on the synthetic sets — and by counting on how many of the 13 real alignments the tested aligner beats the baseline (RANSAC was better in 11 of 13, worse in one case, P2 fraction 00, which had over 6800 rows after alignment).

## Related tools

- **MZmine 2** (Open-source LC-MS processing framework whose peak-list alignment modules (RANSAC aligner and Join aligner) are the objects being scored by the precision/recall benchmark; built with ./gradlew using JDK 12 or newer.) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK)** (Required toolchain for compiling the MZmine 2 source tree so the alignment modules under evaluation can be built and run.)
- **Join aligner** (Baseline alignment algorithm against which the RANSAC aligner's precision and recall are compared on synthetic and real datasets.)

## Examples

```
./gradlew  # build MZmine 2 (JDK 12+) from github.com/mzmine/mzmine2, run RANSAC and Join aligners on the 12 synthetic and P1/P2/M1/M2 datasets, then tabulate precision/recall into ransac_alignment_precision_recall.csv
```

## Evaluation signals

- RANSAC aligner must reach 100% precision and 100% recall on each of the 12 synthetic lipidomic datasets, matching the paper's reported result.
- Per-dataset precision and recall values for both aligners are present in ransac_alignment_precision_recall.csv for all 12 synthetic sets.
- On real data, the tested aligner should outperform the Join aligner in the majority of the 13 alignments (RANSAC did so in 11 of 13).
- Any single regression, such as worse performance on P2 fraction 00 with over 6800 aligned rows, should be identifiable and explainable in the comparison.
- The two aligners must be run on identical input peak lists so precision/recall differences reflect the alignment algorithm rather than upstream processing.

## Limitations

- Developed modules in MZmine 2 were tested mainly on LC-MS data; validation on other data types (e.g. GC-MS) is not demonstrated.
- Synthetic benchmarks are built from seed samples of lipidomic studies, so results may not transfer to other molecule classes or acquisition methods.
- Real-dataset evaluation depends on third-party ground-truth tables (Lange et al.); a single fraction such as P2 00 with over 6800 rows can reverse the ranking between aligners.
- The data-associated ProteomeCommons.org Tranche hash referenced for the manuscript is absent from the provided text, so the exact benchmark files cannot be retrieved from that source.

## Evidence

- [results] RANSAC achieved perfect precision and recall on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] RANSAC beat Join aligner on 11 of 13 real alignments: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] 12 synthetic lipidomic datasets with 20 samples each from one seed sample: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [results] Real datasets with published ground-truth alignments used for evaluation: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [abstract] README confirms synthetic-data evaluation context and repository location: "The performance of the RANSAC alignment was evaluated using synthetic datasets as well as actual experimental data, and the results were compared to those obtained using other alignment algorithms."
- [readme] README build command and JDK requirement for reproducing the alignment modules: "To build the MZmine package from the sources, run the following command: ./gradlew or gradlew.bat"
