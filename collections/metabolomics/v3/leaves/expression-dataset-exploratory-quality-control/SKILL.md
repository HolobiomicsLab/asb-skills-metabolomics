---
name: expression-dataset-exploratory-quality-control
description: Use when a sample-by-gene expression matrix with sample metadata (e.g.
  organ, tumor status) needs QC before downstream analysis, and samples from the same
  biological condition must be checked for consistently high pairwise correlations
  or abnormally low expression.
when_to_use_negative:
- The goal is gene-level differential expression testing (e.g. fold-change ≥ 2 with
  BH-corrected Mann–Whitney U P < 10−3), not sample-level QC
- The dataset has no sample metadata to define biological conditions, so within-condition
  correlation grouping is impossible
- Input is already a cleaned correlation matrix or a feature/gene table with samples
  removed
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: MOG
  role: Computes pairwise Pearson correlations among samples from the same biological
    condition and supports metadata-driven sample filtering and outlier removal; all
    statistical analyses and visualizations were generated exclusively using MOG
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: Java workbench used to load the expression dataset with integrated sample
    metadata, explore samples in the context of metadata, and save the QC exploration
    as a shareable MOG project
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:cff8110ada07f6c20737238a57b648df8e62f84c62d22c69b85e83435cd724e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# expression-dataset-exploratory-quality-control

## Summary

Exploratory quality control of large expression datasets in MetaOmGraph (MOG) by grouping samples with metadata and computing within-condition pairwise Pearson correlations to detect outlier samples. In the Hu-cancer RNA-Seq case study it flagged a single lung adenocarcinoma (LUAD) sample, which was removed so that all remaining within-condition correlations exceeded 0.70.

## When to use

Use when a sample-by-gene expression matrix with sample metadata (e.g. organ, tumor status) needs QC before downstream analysis, and samples from the same biological condition must be checked for consistently high pairwise correlations or abnormally low expression.

## When NOT to use

- The goal is gene-level differential expression testing (e.g. fold-change ≥ 2 with BH-corrected Mann–Whitney U P < 10−3), not sample-level QC
- The dataset has no sample metadata to define biological conditions, so within-condition correlation grouping is impossible
- Input is already a cleaned correlation matrix or a feature/gene table with samples removed

## Inputs

- Sample-by-gene RNA-Seq or microarray expression matrix (e.g. Hu-cancer-RNASeq-dataset with 18,212 genes)
- Sample metadata table with biological condition annotations (organ/cancer type and tumor status)
- MOG project file (.mog) storing the indexed data plus metadata and exploration history

## Outputs

- Sample-sample Pearson correlation matrix (e.g. correlation_matrix.csv)
- Outlier report for flagged samples (e.g. luad_outlier_report.csv)
- Filtered expression dataset with the outlier sample removed

## How to apply

Load the expression dataset (e.g. the Hu-cancer-RNASeq-dataset, 18,212 genes) into MOG as a project with integrated sample metadata, then group samples by biological condition such as cancer type/organ and tumor/non-tumor status using MOG's metadata-driven exploration. Use MOG to compute pairwise Pearson correlations among samples from the same biological condition and screen all within-condition values against the >0.70 acceptance threshold. Flag any sample whose correlations with its condition-mates fall well below 0.70 and inspect it for very low expression values to confirm it is a genuine outlier rather than a labeling error. Remove the confirmed outlier (in the paper's case, one LUAD sample) and recompute the correlation matrix so that all remaining within-condition correlations exceed 0.70. Save the full sample-sample Pearson correlation matrix and a summary report of the flagged outlier for validation and sharing via the savable MOG project.

## Related tools

- **MOG** (Computes pairwise Pearson correlations among samples from the same biological condition and supports metadata-driven sample filtering and outlier removal; all statistical analyses and visualizations were generated exclusively using MOG) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (Java workbench used to load the expression dataset with integrated sample metadata, explore samples in the context of metadata, and save the QC exploration as a shareable MOG project) — https://github.com/urmi-21/MetaOmGraph/

## Evaluation signals

- All remaining within-condition pairwise Pearson correlations exceed the 0.70 threshold after outlier removal
- Exactly one LUAD sample is flagged as the outlier, matching the paper's finding
- The flagged outlier sample shows abnormally low expression values relative to its condition-mates
- correlation_matrix.csv contains the full sample-sample Pearson correlation matrix for every biological condition group
- luad_outlier_report.csv documents the identity of the removed sample and the rationale for exclusion

## Limitations

- The 0.70 correlation threshold is a dataset-specific acceptance criterion from the Hu-cancer RNA-Seq case study and may not transfer to datasets with fewer replicates or noisier platforms
- MOG is a standalone Java application requiring Java Runtime Environment 8 or higher (R 3.4+ optional), so it cannot be scripted headlessly the way R pipelines can
- Correlation-based QC identifies samples deviating from condition-mates but does not by itself distinguish technical batch effects from true biological outliers

## Evidence

- [results] Within-condition correlation QC threshold and outlier finding: [evidence span withheld — anchor status: not_found]
- [results] Workflow step for computing sample correlations in MOG: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [results] Outlier removal filter step: "except one sample from lung adenocarcinoma (LUAD), which we removed from the dataset"
- [abstract] MOG interactive metadata-driven exploration capability: "Researchers, without coding, can interactively visualize and evaluate data in the context of its metadata"
- [readme] MOG prerequisites and installation from README: [evidence span withheld — anchor status: external_doc]
