---
name: ransac-peaklist-alignment
description: Use when multiple deisotoped peak lists from LC-MS runs must be aligned
  across samples to build a consistent feature table, especially when retention-time
  drift or non-linear sample variability degrades simpler join-based alignment.
when_to_use_negative:
- Input is already an aligned feature table or normalized peak list — gap filling
  or normalization is the next stage, not alignment
- Peak lists have not been deisotoped — align ungrouped isotope peaks degrades matching;
  run deisotoping with CDK isotope distributions first
- Evaluating on very dense fractions (e.g. >6800 rows after alignment) where Join
  aligner may outperform RANSAC, as observed on the P2 dataset fraction 00
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Host framework providing the RANSAC peak list alignment module and the alternative
    Join aligner plug-in
  repo: https://github.com/mzmine/mzmine2
- name: mzmine
  role: Successor platform where MZmine development now happens; alignment concept
    continues there
  repo: https://github.com/mzmine/mzmine3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:9b2299a469415bbfed4dc598c15c26eb02a7f9a41bdfe3c93c1bd67392c7ab14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - JFreeChart
  - MZmine 2
  - mzmine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ransac-peaklist-alignment

## Summary

A method for aligning peak lists across LC-MS samples in MZmine 2 using a random sample consensus (RANSAC) algorithm, offered as an alternative to the Join aligner plug-in. It robustly matches peaks across samples by handling retention-time and m/z drift, and was validated against ground-truth alignments from Lange et al. datasets.

## When to use

Use when multiple deisotoped peak lists from LC-MS runs must be aligned across samples to build a consistent feature table, especially when retention-time drift or non-linear sample variability degrades simpler join-based alignment.

## When NOT to use

- Input is already an aligned feature table or normalized peak list — gap filling or normalization is the next stage, not alignment
- Peak lists have not been deisotoped — align ungrouped isotope peaks degrades matching; run deisotoping with CDK isotope distributions first
- Evaluating on very dense fractions (e.g. >6800 rows after alignment) where Join aligner may outperform RANSAC, as observed on the P2 dataset fraction 00

## Inputs

- Deisotoped peak lists from multiple LC-MS samples (post peak picking and isotope grouping)
- Raw MS data files imported into MZmine 2 (optional prerequisite stage)
- Ground-truth alignment tables for benchmarking (e.g. Lange et al. P1, P2, M1, M2)

## Outputs

- An aligned peak list across all samples
- Precision and recall metrics when evaluated against ground-truth alignments
- An intermediate peak list file (MZmine 2 never removes original unprocessed data)

## How to apply

After importing raw MS data (preserving full mass precision), optionally filtering, peak picking, and deisotoping peak lists via CDK-calculated isotope distributions, apply the RANSAC peak list aligner to align lists across samples; the Join aligner remains available as an alternative plug-in. Evaluate alignment quality against 'ground truth' alignment tables (e.g. the P1, P2, M1, M2 proteomic and metabolomic datasets from Lange et al.) using precision and recall. On 12 synthetic lipidomic datasets (20 samples each, seeded from real lipidomic studies), RANSAC achieved 100% precision and recall, and outperformed Join aligner in 11 of 13 alignments. Expect some row inflation in dense datasets — e.g. the P2 fraction 00 alignment produced over 6800 rows, the one case where RANSAC was worse than Join aligner.

## Related tools

- **MZmine 2** (Host framework providing the RANSAC peak list alignment module and the alternative Join aligner plug-in) — https://github.com/mzmine/mzmine2
- **mzmine** (Successor platform where MZmine development now happens; alignment concept continues there) — https://github.com/mzmine/mzmine3

## Evaluation signals

- Precision and recall of the aligned peak list against ground-truth alignment tables; RANSAC reached 100% in both on the 12 synthetic lipidomic datasets
- Comparative check: RANSAC should beat Join aligner in roughly 11 of 13 alignments; a worse result on a dense fraction (like P2 fraction 00, >6800 rows) is a known exception, not a bug
- The original unprocessed peak lists must remain untouched — MZmine 2 never removes original unprocessed data, so verify each intermediate peak list exists as a separate file
- Aligned output must precede gap filling and normalization; a correctly aligned list feeds the gap-filling module and then export as normalized_peak_list.csv

## Limitations

- RANSAC alignment was worse than Join aligner in a single case (P2 dataset fraction 00, over 6800 rows after alignment), so row density can flip the choice of aligner
- Developed modules were tested mainly on LC-MS data only; validation on other data types is not demonstrated in the paper
- No specific framework version number is stated, so reproducibility of the exact aligner behavior may require pinning a source revision (mzmine2 is archived; development moved to mzmine3)

## Evidence

- [abstract] RANSAC performance vs other aligners: "The performance of the RANSAC alignment was evaluated using synthetic datasets as well as actual experimental data, and the results were compared to those obtained using other alignment algorithms."
- [results] RANSAC 100% precision and recall on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] RANSAC beats Join aligner in 11 of 13 alignments: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Workflow stage: alignment precedes gap filling and normalization: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
- [readme] Tool scope: complete MS data analysis workflow modules: "MZmine 2 is an open-source software for mass-spectrometry data processing. The goals of the project is to provide a user-friendly, flexible and easily extendable software with a complete set of"
