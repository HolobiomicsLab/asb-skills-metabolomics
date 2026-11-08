---
name: nonparametric-significance-testing-mann-whitney
description: Use when comparing a gene's expression distributions between two independent,
  unpaired sample groups (e.g. LIHC tumor vs non-tumor) from an expression matrix
  and metadata, without assuming normality of expression values.
when_to_use_negative:
- When the two groups are paired or matched samples, where a nonparametric paired
  test rather than the unpaired Mann–Whitney U test is appropriate
- When the goal is comparing more than two groups or continuous covariates rather
  than exactly two categorical sample groups
- When expression values cannot be meaningfully log2 transformed or the contrast is
  between conditions of different organs, which MOG analyses separated first
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MetaOmGraph
  role: Runs the Mann–Whitney U test on log2 transformed expression data between sample
    groups, with BH correction, using metadata to separate tumor/non-tumor or stage
    groups
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional environment to which MOG data subsets can be sent for additional
    statistical analyses
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:a33306e6854191c6550321cdf5329e81fcef16b1b11e4fb26b5a0f05ed7d4771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nonparametric-significance-testing-mann-whitney

## Summary

Applies the Mann–Whitney U test on log2-transformed RNA-Seq/microarray expression values to identify genes differentially expressed between two sample groups (e.g. tumor vs non-tumor in the Hu-cancer-RNASeq-dataset), with Benjamini–Hochberg correction at P < 10^-3 as used in MetaOmGraph (MOG).

## When to use

Use when comparing a gene's expression distributions between two independent, unpaired sample groups (e.g. LIHC tumor vs non-tumor) from an expression matrix and metadata, without assuming normality of expression values.

## When NOT to use

- When the two groups are paired or matched samples, where a nonparametric paired test rather than the unpaired Mann–Whitney U test is appropriate
- When the goal is comparing more than two groups or continuous covariates rather than exactly two categorical sample groups
- When expression values cannot be meaningfully log2 transformed or the contrast is between conditions of different organs, which MOG analyses separated first

## Inputs

- Large expression matrix (e.g. Hu-cancer-RNASeq-dataset with 18,212 genes) loaded as a MOG project
- Sample metadata annotating tumor status, organ and stage (e.g. from PMR)
- Group assignment defining the two samples classes to compare (tumor vs non-tumor, or early vs late stage)

## Outputs

- Per-gene BH-corrected Mann–Whitney U test P-values and fold-changes
- List of differentially expressed genes meeting BH-corrected P < 10^-3 and fold-change >= 2
- Table of putative biomarker rows (e.g. lihc_biomarker_expression_table.csv) with gene identifier, tumor expression, non-tumor expression, fold-change and corrected P-value

## How to apply

Load the expression dataset and sample metadata (tumor status, organ) into MetaOmGraph (MOG), a Java workbench that can send subsets to R for additional analyses. Filter to the samples of interest (e.g. LIHC samples separated into tumor and non-tumor groups by metadata), then log2 transform the expression values — MOG runs the Mann–Whitney U test on the log2 transformed data because expression distributions are skewed and non-normal. Run the test per gene between the two groups, apply Benjamini–Hochberg correction over all tested genes, and call a gene significant when the BH-corrected P-value < 10^-3. Further require a fold-change ≥ 2 between groups to nominate differentially expressed genes; in the article's LIHC case this surfaced downregulated putative biomarkers such as ARG1, CYP2C8, CYP3A4, CYP3A7 and CYP4A11. The same procedure applies to stage-stratified comparisons (early stage I/II vs late stage III+) after separating samples by organ.

## Related tools

- **MetaOmGraph** (Runs the Mann–Whitney U test on log2 transformed expression data between sample groups, with BH correction, using metadata to separate tumor/non-tumor or stage groups) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional environment to which MOG data subsets can be sent for additional statistical analyses)

## Examples

```
wt <- wilcox.test(log2(expr) ~ group, data=lihc); padj <- p.adjust(wt$p.value, method="BH")
```

## Evaluation signals

- Output table contains fields for gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value
- Every called marker satisfies both fold-change >= 2 and BH-corrected P < 10^-3, matching the paper's differential expression criteria
- Test was performed on log2 transformed values, not raw expression
- Sample groups were defined from metadata (e.g. LIHC tumor vs non-tumor), and QC checked via pairwise Pearson correlations > 0.70 within condition (outlier samples such as one LUAD sample removed)
- In the LIHC case, the reported downregulated biomarkers (ARG1, CYP2C8, CYP3A4, CYP3A7, CYP4A11) are recoverable from the output

## Limitations

- The article reports that in each cancer type of the Hu-cancer-RNASeq-dataset between 2000 and 5000 of the 18,212 genes are differentially expressed, so significance alone does not imply biomarker relevance — fold-change filtering and external validation (e.g. The Human Protein Atlas prognostic markers) are needed
- One LUAD sample with Pearson correlation <= 0.70 had to be removed, so batch effects or outlier samples can distort group tests if not checked first
- MOG requires Java Runtime Environment 8 or higher, and R integration is optional (R 3.4 or higher)

## Evidence

- [results] Mann–Whitney U test on log2 transformed data with BH threshold: "Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3)"
- [abstract] MOG statistical analyses include significance tests: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [abstract] MOG can send data subsets to R: "Researchers can send data subsets to R for additional analyses."
- [readme] MOG interactive exploration with metadata on local data: "By incorporating metadata, MOG adds another dimension to the analyses and provides flexibility in data exploration. MOG allows users to explore their own data on their local machines."
