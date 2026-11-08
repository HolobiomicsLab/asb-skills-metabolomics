---
name: rna-seq-sample-outlier-detection
description: Use when an RNA-Seq or expression dataset with sample metadata (condition,
  organ, tumor status) needs a QC pass to confirm that within-condition samples are
  consistent and to flag outlier samples before differential expression analysis.
when_to_use_negative:
- The dataset has no sample metadata to define biological conditions, so within-condition
  correlation groups cannot be formed.
- The goal is gene-level (not sample-level) outlier screening, e.g. identifying outlier
  genes like CDKN2A via Spearman correlation < 0.50.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Compute pairwise Pearson correlations among samples from the same biological
    condition to screen for outliers and batch effects
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: Standalone Java workbench used exclusively to generate the statistical analyses
    and visualizations for the sample QC, including the correlation matrix
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:553e2da7898aa053e4b169a6096fc9ef95860aacb71f41651ba136d70674001a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# rna-seq-sample-outlier-detection

## Summary

QC skill for RNA-Seq expression matrices that computes pairwise Pearson correlations among samples grouped by biological condition (e.g. organ and tumor status) to detect samples with abnormally low within-condition correlations and very low expression values, as demonstrated on the Hu-cancer-RNASeq-dataset in MetaOmGraph (MOG).

## When to use

Use when an RNA-Seq or expression dataset with sample metadata (condition, organ, tumor status) needs a QC pass to confirm that within-condition samples are consistent and to flag outlier samples before differential expression analysis.

## When NOT to use

- The dataset has no sample metadata to define biological conditions, so within-condition correlation groups cannot be formed.
- The goal is gene-level (not sample-level) outlier screening, e.g. identifying outlier genes like CDKN2A via Spearman correlation < 0.50.

## Inputs

- Hu-cancer-RNASeq-dataset MOG project (RNA-Seq expression matrix, 18 212 genes)
- Sample metadata (cancer type / organ, tumor status) integrated in the MOG project

## Outputs

- correlation_matrix.csv (full sample-sample pairwise Pearson correlation matrix)
- luad_outlier_report.csv (summary of the flagged LUAD outlier sample)
- Filtered expression dataset with the outlier sample removed

## How to apply

Load the expression dataset (e.g. the Hu-cancer RNA-Seq MOG project, 18 212 genes) together with its sample metadata in MetaOmGraph, and group samples by biological condition (cancer type/organ and tumor status). Use MOG to compute pairwise Pearson correlations among samples from the same biological condition and screen all within-condition correlation values against the acceptance threshold of >0.70. Flag any sample whose correlations with its condition-mates fall well below 0.70, and inspect its expression values for abnormally very low signal to confirm it is a technical outlier rather than biology. In the Hu-cancer case study this procedure identified exactly one lung adenocarcinoma (LUAD) sample, which was removed from the dataset. Save the full sample-sample Pearson correlation matrix and a report of the flagged outlier, and validate that all remaining within-condition pairwise correlations exceed 0.70 with exactly one sample flagged.

## Related tools

- **MetaOmGraph** (Compute pairwise Pearson correlations among samples from the same biological condition to screen for outliers and batch effects) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (Standalone Java workbench used exclusively to generate the statistical analyses and visualizations for the sample QC, including the correlation matrix) — https://github.com/urmi-21/MetaOmGraph/

## Evaluation signals

- All remaining within-condition pairwise Pearson correlations must exceed the 0.70 threshold after outlier removal.
- Exactly one LUAD sample is flagged as the outlier (matching the published finding of one lung adenocarcinoma sample removed).
- The flagged sample also shows very low expression values, corroborating that low correlation reflects poor sample quality rather than biology.
- correlation_matrix.csv contains a complete sample-sample Pearson correlation matrix and luad_outlier_report.csv documents the flagged sample.
- The saved MOG project preserves the exploration history so the QC can be reproduced or shared.

## Limitations

- The 0.70 within-condition Pearson correlation threshold was calibrated on the Hu-cancer-RNASeq-dataset; other datasets may require re-justified thresholds.
- Only samples whose correlations fall 'well below' the 0.70 threshold are flagged; borderline cases require manual judgment.
- MOG is specialized for expression data and interactive exploratory analysis; large-scale automated pipelines may prefer scripted alternatives.

## Evidence

- [results] Sample QC via within-condition Pearson correlation and LUAD outlier removal: [evidence span withheld — anchor status: not_found]
- [results] Correlation computation method: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [readme] Tool grounding for the analysis: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets. MOG overcomes the challenges posed by big size and complexity of big datasets"
- [abstract] Save and share reproducibility: "MOG projects, with history of explorations, can be saved and shared."
