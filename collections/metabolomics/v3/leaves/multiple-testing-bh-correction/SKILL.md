---
name: multiple-testing-bh-correction
description: Use when thousands of genes are tested simultaneously for differential
  expression between sample groups (e.g. tumor vs non-tumor) and raw Mann–Whitney
  U or other per-gene P-values must be corrected for multiple testing before filtering.
when_to_use_negative:
- Only a handful of pre-defined genes are tested with no simultaneous-testing inflation
  — a raw P-value suffices and BH correction would over-penalize
- The comparison is not gene-wise hypothesis testing (e.g. sample-level batch-effect
  checks by pairwise Pearson correlation), which MOG performs without a correction
  step
- Expression values are not comparable across groups (no log2 transform applied),
  since the paper's test and correction are defined on log2-transformed data
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: MetaOmGraph
  role: Runs the Mann–Whitney U test on log2-transformed data and applies BH correction
    internally as part of its differential expression analysis, without coding
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional integration for sending data subsets to R for additional statistical
    analyses such as P-value adjustment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:33a1c3c151bb19e624d5c02d91d12d51b656c7209a9c8a3321d568ac54222503
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# multiple-testing-bh-correction

## Summary

Apply Benjamini–Hochberg false-discovery-rate correction to per-gene P-values from differential expression tests (e.g. Mann–Whitney U tests on log2-transformed RNA-Seq data), so that gene-level calls are evaluated against a strict BH-corrected P-value threshold (<10^-3) rather than raw P-values.

## When to use

Use when thousands of genes are tested simultaneously for differential expression between sample groups (e.g. tumor vs non-tumor) and raw Mann–Whitney U or other per-gene P-values must be corrected for multiple testing before filtering.

## When NOT to use

- Only a handful of pre-defined genes are tested with no simultaneous-testing inflation — a raw P-value suffices and BH correction would over-penalize
- The comparison is not gene-wise hypothesis testing (e.g. sample-level batch-effect checks by pairwise Pearson correlation), which MOG performs without a correction step
- Expression values are not comparable across groups (no log2 transform applied), since the paper's test and correction are defined on log2-transformed data

## Inputs

- Raw per-gene P-values from a two-group statistical test (e.g. Mann–Whitney U test) over all genes in the expression matrix
- Log2-transformed expression dataset (e.g. Hu-cancer RNA-Seq matrix with 18,212 genes)
- Sample metadata defining the two comparison groups (e.g. tumor status and organ, LIHC tumor vs non-tumor)

## Outputs

- BH-corrected P-values per gene
- List of significant differentially expressed genes satisfying BH-corrected P < 10^-3 and fold-change >= 2
- Biomarker table with gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value (e.g. lihc_biomarker_expression_table.csv)

## How to apply

Run the per-gene statistical test first — in the MetaOmGraph (MOG) workflow, a Mann–Whitney U test is computed on log2-transformed expression values between the two groups (e.g. LIHC tumor vs non-tumor samples). Collect the raw P-values for all tested genes (18,212 genes in the Hu-cancer-RNASeq-dataset) and apply Benjamini–Hochberg correction across the full gene set, e.g. via R's p.adjust(method="BH") when using MOG's optional R integration. Call a gene differentially expressed only if the BH-corrected P-value is <10^-3 AND the fold-change between groups is >= 2. Rationale: the paper deliberately couples a very stringent corrected-P cutoff with a fold-change filter to yield robust biomarker candidates (e.g. ARG1, CYP2C8, GPC3, CYP2A6 in LIHC); MOG computes and applies this correction internally so researchers can explore results without coding.

## Related tools

- **MetaOmGraph** (Runs the Mann–Whitney U test on log2-transformed data and applies BH correction internally as part of its differential expression analysis, without coding) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional integration for sending data subsets to R for additional statistical analyses such as P-value adjustment)

## Examples

```
pvals_bh <- p.adjust(raw_pvals, method = "BH")  # MOG applies BH correction internally; R integration requires R 3.4+
```

## Evaluation signals

- Output table contains fields for gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value
- Every reported marker satisfies fold-change >= 2 AND BH-corrected P < 10^-3, matching the paper's differential expression criteria
- Correction was applied across the full tested gene set (e.g. all 18,212 genes), not per-gene independently
- Number of differentially expressed genes per cancer type falls in the expected range (2000–5000 of 18,212 genes in the Hu-cancer-RNASeq-dataset)
- Corrected P-values are monotonically >= raw P-values and none decrease after adjustment (sanity invariant of BH FDR correction)

## Limitations

- The very strict BH-corrected threshold of 10^-3 combined with fold-change >= 2 may exclude real but modest expression differences
- Results depend on prior QC — in the paper one LUAD sample with Pearson correlation below the >0.70 threshold had to be removed before testing
- MOG requires a Java Runtime Environment 8 (or higher); the R link is optional (R 3.4+), so correction steps are only available in-MOG or via optional R

## Evidence

- [results] Identify differentially expressed genes with fold-change >= 2 and BH-corrected Mann–Whitney U test P < 10−3: "Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3)"
- [abstract] MOG includes significance tests for differential expression analysis: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [abstract] MOG sends data subsets to R for additional analyses: "Researchers can send data subsets to R for additional analyses."
- [readme] MOG is specialized for expression data and supports statistical analysis: [evidence span withheld — anchor status: external_doc]
