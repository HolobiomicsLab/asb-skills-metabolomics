---
name: supplementary-table-result-verification
description: Use when benchmarking candidate gene prioritization output against a
  reference causal gene set, or when reproducing/validating enrichment statistics
  (odds ratios with 95% CIs) against published supplementary tables such as ProGeM's
  Table S7.
when_to_use_negative:
- Input is a cis-pQTL or trans-QTL dataset rather than the 227 curated mQTL reference
  set — the reference causal assignments and reported ORs (29/45/84/169) are mQTL-specific
  and ProGeM's top-down strategy is not directly applicable to trans-pQTLs
- You need effect-size enrichment of continuous scores (e.g. top-down metabolic database
  score out of 5) rather than a binary candidate/causal 2x2 contingency analysis
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: ProGeM
  role: Generates the bottom-up, top-down, cooccurring, and nearest-gene candidate
    output tables that are cross-tabulated against reference causal genes for Fisher's
    exact testing
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Environment for loading candidate tables with data.table, constructing the
    2x2 contingency tables, and running stats::fisher.test to obtain odds ratios and
    95% CIs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:19c3187e950484f9f471e1aa1c66ecac4e13c5ecbc78a4b870a72d317aa045db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# supplementary-table-result-verification

## Summary

Recomputes the Fisher's exact test enrichment of ProGeM candidate gene definitions against reference causal genes at the 227 curated mQTLs and verifies that the recomputed odds ratios (concurrent OR=29, nearest-gene OR=45, nearest-concurrent OR=84, moderate-impact OR=169) match the values reported in Supplementary Table S7 and plotted in Figure 3.

## When to use

Use when benchmarking candidate gene prioritization output against a reference causal gene set, or when reproducing/validating enrichment statistics (odds ratios with 95% CIs) against published supplementary tables such as ProGeM's Table S7.

## When NOT to use

- Input is a cis-pQTL or trans-QTL dataset rather than the 227 curated mQTL reference set — the reference causal assignments and reported ORs (29/45/84/169) are mQTL-specific and ProGeM's top-down strategy is not directly applicable to trans-pQTLs
- You need effect-size enrichment of continuous scores (e.g. top-down metabolic database score out of 5) rather than a binary candidate/causal 2x2 contingency analysis

## Inputs

- ProGeM output tables: OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt, OUTPUT_nearest_genes.txt
- Reference causal gene assignments for the 227 curated metabolite QTLs (Supplementary Table S1 mQTL reference dataset)

## Outputs

- fisher_exact_odds_ratios.tsv containing odds ratios and 95% confidence intervals for the concurrent (OR=29), nearest-gene (OR=45), nearest-concurrent (OR=84), and moderate-impact (OR=169) candidate sets

## How to apply

Load the four ProGeM candidate output tables (OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt, OUTPUT_nearest_genes.txt) with data.table in R, then load the reference causal gene assignments for the 227 curated mQTLs from Supplementary Table S1. For each candidate definition (concurrent, nearest-gene, nearest-concurrent, and moderate/high VEP IMPACT sentinel variant sets), build a 2x2 contingency table crossing candidate-vs-non-candidate against reference-causal-vs-non-causal gene assignments across the 227 loci. Run stats::fisher.test on each table to obtain the odds ratio and 95% confidence interval, and write all four odds ratios with CIs to fisher_exact_odds_ratios.tsv. Validation is by direct comparison to the reported enrichment values: concurrent OR=29, nearest-gene OR=45 [31-67] with P = 1.0 x 10^-107, nearest-concurrent OR=84, and moderate-impact OR=169, as tabulated in Supplementary Table S7 and plotted in Figure 3.

## Related tools

- **ProGeM** (Generates the bottom-up, top-down, cooccurring, and nearest-gene candidate output tables that are cross-tabulated against reference causal genes for Fisher's exact testing) — https://github.com/ds763/ProGeM
- **R** (Environment for loading candidate tables with data.table, constructing the 2x2 contingency tables, and running stats::fisher.test to obtain odds ratios and 95% CIs)

## Examples

```
bt <- fread('OUTPUT_bottom_up_summary.txt'); ft <- fisher.test(table(candidate=bt$nearest==1, causal=gene_is_reference_causal)); ft$estimate; ft$conf.int; ft$p.value
```

## Evaluation signals

- Recomputed nearest-gene odds ratio equals 45 with 95% CI [31-67] and P = 1.0 x 10^-107, matching Supplementary Table S7 / Figure 3
- Recomputed concurrent, nearest-concurrent, and moderate-impact odds ratios equal 29, 84, and 169 respectively
- All contingency tables are constructed over exactly the 227 curated metabolite QTL loci
- fisher_exact_odds_ratios.tsv contains all four candidate definitions each with an odds ratio and 95% confidence interval
- Sensitivity cross-check: 98% of the 227 loci have the expert-curated gene among ProGeM candidates (only 4 of 227 reference causal genes missed)

## Limitations

- The reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as the prioritization of the nearest genes, which may inflate the nearest-gene odds ratio
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed sensitivity at mQTLs may be inflated
- Whether genomic proximity is a good indicator of a true positive causal gene for other molecular QTLs requires further research and depends on the availability of high-confidence reference datasets

## Evidence

- [abstract] nearest-gene enrichment: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [discussion] reference dataset availability: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
- [readme] moderate-impact candidate definition: "ProGeM will highlight any bottom-up candidate mediating genes that contain either a sentinel or a proxy variant expected to exert a HIGH or a MODERATE IMPACT"
- [intro] benchmarking scope: "we have also determined the sensitivity and specificity of ProGeM using two molecular QTL datasets, comprising 227 mQTLs and 562 cis-pQTLs, for which each QTL has been assigned a high-confidence"
- [abstract] cis-eQTL enrichment caveat: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
