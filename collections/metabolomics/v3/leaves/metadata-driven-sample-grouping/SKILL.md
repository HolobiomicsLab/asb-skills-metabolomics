---
name: metadata-driven-sample-grouping
description: Use when an expression dataset carries sample metadata (organ, tumor/non-tumor
  status, disease stage) and analyses — correlation QC, Mann–Whitney U tests, outlier
  screening — must be stratified by biological condition rather than run across all
  samples pooled.
when_to_use_negative:
- The dataset lacks sample metadata or metadata cannot be integrated into the project,
  so groups cannot be defined by organ/tumor status
- The goal is a pooled, whole-dataset analysis (e.g. globally consistent DE genes
  across all cancers) where condition-level separation would change the question
- Input is single-condition data with no tumor/non-tumor or stage contrast to stratify
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: MOG
  role: Interactive grouping and filtering of samples by integrated metadata; compute
    within-condition pairwise Pearson correlations for QC and define groups for Mann–Whitney
    U tests
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: The MOG workbench application used to visualize data in the context of metadata
    and save/share projects with the exploration history
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:dbdb6b99995b6ab45f199f8672d3ee0807b90e53cdc706ad9b63a58119379f55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metadata-driven-sample-grouping

## Summary

Group expression-dataset samples by biological condition using integrated sample metadata (e.g. organ/cancer type and tumor status in the Hu-cancer-RNASeq-dataset) before downstream QC and statistical tests in MetaOmGraph (MOG). Metadata-driven grouping underpins pairwise sample-correlation QC, outlier detection, and group-wise differential expression analysis.

## When to use

Use when an expression dataset carries sample metadata (organ, tumor/non-tumor status, disease stage) and analyses — correlation QC, Mann–Whitney U tests, outlier screening — must be stratified by biological condition rather than run across all samples pooled.

## When NOT to use

- The dataset lacks sample metadata or metadata cannot be integrated into the project, so groups cannot be defined by organ/tumor status
- The goal is a pooled, whole-dataset analysis (e.g. globally consistent DE genes across all cancers) where condition-level separation would change the question
- Input is single-condition data with no tumor/non-tumor or stage contrast to stratify

## Inputs

- RNA-Seq expression matrix from the Hu-cancer-RNASeq-dataset (18,212 genes) as a MOG project
- Sample metadata table with organ/cancer type, tumor status, and optional stage annotations

## Outputs

- Condition-grouped sample sets (e.g. tumor vs non-tumor per organ) ready for correlation QC or differential expression
- Within-condition pairwise Pearson correlation matrix (saved as correlation_matrix.csv)
- Flagged outlier report (e.g. luad_outlier_report.csv for the removed LUAD sample)

## How to apply

Load the expression matrix together with its sample metadata into MOG as a project, so samples can be filtered and grouped interactively by metadata fields such as cancer type/organ and tumor status. Separate samples into condition groups (e.g. LIHC tumor vs non-tumor, or organ-wise early stage I/II vs late stage III+). Within each condition group, use MOG to compute pairwise Pearson correlations and screen against a threshold of >0.70 to confirm sample consistency; flag any sample whose correlations with condition-mates fall well below 0.70 and check it for very low expression values, as done for the single LUAD outlier that was removed. Apply the same grouping before running Mann–Whitney U tests on log2-transformed values (BH corrected P < 10^-3, fold-change ≥ 2) so differential expression is computed between, not within, biological conditions.

## Related tools

- **MOG** (Interactive grouping and filtering of samples by integrated metadata; compute within-condition pairwise Pearson correlations for QC and define groups for Mann–Whitney U tests) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (The MOG workbench application used to visualize data in the context of metadata and save/share projects with the exploration history) — https://github.com/urmi-21/MetaOmGraph/

## Evaluation signals

- All remaining within-condition pairwise Pearson correlations exceed the 0.70 threshold after grouping
- Exactly one LUAD sample is flagged as the outlier (correlations well below 0.70 with condition-mates, very low expression values)
- Each defined group is internally homogeneous by organ and tumor status in the metadata
- The correlation matrix CSV contains pairwise Pearson values for every sample pair within each condition
- Downstream Mann–Whitney U comparisons are run only between metadata-defined groups, matching the paper's tumor vs non-tumor contrast

## Limitations

- Grouping quality depends entirely on correct, complete sample metadata; mislabeled organ or tumor-status fields will corrupt condition groups
- A single misbehaving sample (as with the LUAD outlier) can distort within-condition correlation statistics and must be removed before downstream tests
- MOG requires Java Runtime Environment 8+ (R 3.4+ optional), so the interactive metadata-driven workflow is tied to the MOG desktop application

## Evidence

- [results] All samples had high within-condition pairwise Pearson correlations (>0.70) with others from the same organ and tumor status, except one lung adenocarcinoma (LUAD) sample, which was removed from the dataset as an outlier.: [evidence span withheld — anchor status: not_found]
- [results] MOG computes correlations among samples sharing metadata-defined biological conditions to detect batch effects and outliers.: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [abstract] MOG's metadata integration lets researchers interactively visualize and evaluate data in the context of its metadata without coding.: "Researchers, without coding, can interactively visualize and evaluate data in the context of its metadata"
- [readme] The MOG executable requires Java 8+ and optionally R 3.4+ for sending data subsets for additional analyses.: "* Java Runtime Environment 8 (or higher)
* R 3.4 (or higher) [optional]"
- [results] Samples can be separated by metadata fields such as organ and stage to define comparison groups.: "We used MOG to separate samples by organ, and then by early stage (stage I or stage II) and late stage (stage III and later)"
