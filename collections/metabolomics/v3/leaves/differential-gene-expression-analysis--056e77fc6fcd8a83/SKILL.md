---
name: differential-gene-expression-analysis
description: Use when a gene expression matrix (e.g., RNA-Seq counts or microarray
  intensities) has two metadata-defined sample groups and the goal is to flag genes
  significantly up- or down-regulated between them, as in tumor vs non-tumor LIHC
  comparison.
when_to_use_negative:
- The comparison involves more than two groups or a continuous covariate, where MOG's
  two-group Mann–Whitney U test design does not apply directly
- The goal is co-expression module discovery rather than group contrast — use MOG's
  co-expression network analysis with MCODE in Cytoscape instead
- Input data are already normalized log-transformed values with a prior DE analysis
  result; re-running the log2 transform and test would be redundant
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: MOG
  role: Performs the log2 transform, Mann–Whitney U test, BH correction, fold-change
    filtering, and interactive visualization (box plots, volcano plots) exclusively
    within the tool
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: Standalone Java workbench used to load the expression dataset with metadata,
    define sample groups, and save/share the analysis as a MOG project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: 'Optional: data subsets can be sent from MOG to R for additional analyses'
- name: The Human Protein Atlas (THPA)
  role: External resource used to flag differentially expressed genes as prognostic
    markers (e.g., ARG1, CYP2C8, CYP3A4, CYP3A7, CYP4A11 in LIHC)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:9fc24d18246caaeac77f5e9b7f780a02454770dddff3e17030e25b346c505cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  - The Human Protein Atlas (THPA)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# differential-gene-expression-analysis

## Summary

Identify differentially expressed genes between two sample groups (e.g., tumor vs non-tumor) in large expression datasets using MetaOmGraph's Mann–Whitney U test on log2-transformed values with Benjamini–Hochberg correction, followed by a fold-change filter.

## When to use

Use when a gene expression matrix (e.g., RNA-Seq counts or microarray intensities) has two metadata-defined sample groups and the goal is to flag genes significantly up- or down-regulated between them, as in tumor vs non-tumor LIHC comparison.

## When NOT to use

- The comparison involves more than two groups or a continuous covariate, where MOG's two-group Mann–Whitney U test design does not apply directly
- The goal is co-expression module discovery rather than group contrast — use MOG's co-expression network analysis with MCODE in Cytoscape instead
- Input data are already normalized log-transformed values with a prior DE analysis result; re-running the log2 transform and test would be redundant

## Inputs

- Large expression matrix (e.g., Hu-cancer-RNASeq-dataset RNA-Seq, or AT-microarray-dataset)
- Sample metadata table with tumor status, organ, and/or disease stage
- Group definition (e.g., LIHC tumor vs non-tumor, or early vs late stage)

## Outputs

- List of differentially expressed genes with fold-change and BH-corrected Mann–Whitney U test P-values (e.g., lihc_biomarker_expression_table.csv with gene identifier, tumor expression, non-tumor expression, fold-change, corrected P-value)
- Upregulated / downregulated gene sets for downstream GO enrichment or co-expression network analysis

## How to apply

Load the expression dataset and its sample metadata (tumor status, organ, stage) into MetaOmGraph as a project, then use the metadata to split samples into the two comparison groups (e.g., LIHC tumor vs non-tumor). Log2 transform the expression values and run MOG's Mann–Whitney U test between groups, applying Benjamini–Hochberg correction with a significance threshold of BH-corrected P-value < 10^-3. Filter the significant genes to those with fold-change ≥ 2 between the groups; in the Hu-cancer-RNASeq-dataset this yields 2000–5000 of 18,212 genes per cancer type. Optionally cross-reference flagged genes with external resources such as The Human Protein Atlas (THPA) to identify putative prognostic biomarkers, and check pairwise Pearson correlations (>0.70) among same-condition samples beforehand to detect batch effects or outlier samples (e.g., one LUAD sample was removed).

## Related tools

- **MOG** (Performs the log2 transform, Mann–Whitney U test, BH correction, fold-change filtering, and interactive visualization (box plots, volcano plots) exclusively within the tool) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (Standalone Java workbench used to load the expression dataset with metadata, define sample groups, and save/share the analysis as a MOG project) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional: data subsets can be sent from MOG to R for additional analyses)
- **The Human Protein Atlas (THPA)** (External resource used to flag differentially expressed genes as prognostic markers (e.g., ARG1, CYP2C8, CYP3A4, CYP3A7, CYP4A11 in LIHC))

## Evaluation signals

- Output table contains required fields: gene identifier, tumor expression, non-tumor expression, fold-change, and BH-corrected P-value
- Every reported marker satisfies fold-change ≥ 2 AND BH-corrected P < 10^-3
- Number of differentially expressed genes falls within the expected 2000–5000 range out of 18,212 genes per cancer type in the Hu-cancer-RNASeq-dataset
- Same-condition sample pairwise Pearson correlations are >0.70 after removing outliers (e.g., one anomalous LUAD sample)
- Direction of change is consistent with metadata group definitions (e.g., ARG1, CYP2C8, CYP3A4, CYP3A7, CYP4A11 decrease with LIHC progression)

## Limitations

- One outlier sample (LUAD) with Pearson correlation below the >0.70 within-condition threshold had to be removed before analysis
- The Mann–Whitney U test is run on log2-transformed data; untransformed or improperly normalized input may invalidate the significance calls
- MOG is a Java desktop application requiring Java Runtime Environment 8+; R 3.4+ integration is optional and not required for the core test

## Evidence

- [results] Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3): "Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3)"
- [results] In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [results] we used MOG to compute pairwise Pearson correlations among samples from the same biological condition: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [results] The statistical analyses and visualizations shown were generated exclusively using MOG.: "The statistical analyses and visualizations shown were generated exclusively using MOG."
- [abstract] Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests.: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
