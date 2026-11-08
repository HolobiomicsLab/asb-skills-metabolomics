---
name: candidate-gene-set-contingency-analysis
description: Use when benchmarking gene-prioritization criteria against a reference
  set of curated causal genes at QTL loci and you need per-criterion odds ratios with
  95% confidence intervals from 2x2 contingency tables.
when_to_use_negative:
- Your reference causal gene set is not independent of the prioritization tool (e.g.
  KEGG guided both ProGeM's top-down component and the expert literature review),
  so enrichment/sensitivity estimates may be inflated and this contingency benchmark
  would overstate performance
- 'You are evaluating trans-pQTL loci: ProGeM''s top-down strategy and its candidate
  definitions are not directly applicable there and would require a different approach'
- You need candidate scoring/ranking within a locus rather than aggregate enrichment
  of candidate definitions across loci
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Generates the candidate causal gene output tables (bottom-up summary, top-down
    scores, concurrent, nearest genes) that are cross-tabulated against reference
    causal genes
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Used with data.table to load output tables and stats::fisher.test to compute
    odds ratios and 95% confidence intervals per candidate definition
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:77e2c01db6fc037aedd5ac295d4677a8b96847689a0a6a679251fbed369158af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# candidate-gene-set-contingency-analysis

## Summary

Quantify how enriched each ProGeM candidate causal gene definition (bottom-up nearest-gene, concurrent, moderate-impact sentinel variant, etc.) is with reference 'true positive' causal genes at the 227 curated mQTLs by building 2x2 contingency tables (candidate vs non-candidate x reference-causal vs non-causal) and running Fisher's exact test in R.

## When to use

Use when benchmarking gene-prioritization criteria against a reference set of curated causal genes at QTL loci and you need per-criterion odds ratios with 95% confidence intervals from 2x2 contingency tables.

## When NOT to use

- Your reference causal gene set is not independent of the prioritization tool (e.g. KEGG guided both ProGeM's top-down component and the expert literature review), so enrichment/sensitivity estimates may be inflated and this contingency benchmark would overstate performance
- You are evaluating trans-pQTL loci: ProGeM's top-down strategy and its candidate definitions are not directly applicable there and would require a different approach
- You need candidate scoring/ranking within a locus rather than aggregate enrichment of candidate definitions across loci

## Inputs

- OUTPUT_bottom_up_summary.txt (tab-separated ProGeM bottom-up candidate annotation with nearest/second_nearest/third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT boolean/nominal columns)
- OUTPUT_top_down_scores.txt (ProGeM top-down candidates with GO/KEGG/MGI/orphanet/reactome booleans and database score out of 5)
- OUTPUT_cooccurring_candidates.txt (concurrent bottom-up AND top-down candidates)
- OUTPUT_nearest_genes.txt (genes nearest to each sentinel variant)
- Supplementary Table S1 mQTL reference dataset: expert-curated causal gene assignments for 227 metabolite QTLs

## Outputs

- fisher_exact_odds_ratios.tsv: odds ratios with 95% confidence intervals for the concurrent, nearest-gene, nearest-concurrent, and moderate-impact sentinel variant candidate sets
- Per-criterion 2x2 contingency tables (candidate-vs-non-candidate x reference-causal-vs-non-causal) across the 227 loci

## How to apply

Load the ProGeM candidate gene output tables ('OUTPUT_bottom_up_summary.txt', 'OUTPUT_top_down_scores.txt', 'OUTPUT_cooccurring_candidates.txt', 'OUTPUT_nearest_genes.txt') with data.table in R, along with the reference causal gene assignments from Supplementary Table S1 for the 227 curated metabolite QTLs. For each candidate definition, cross candidate-vs-non-candidate against reference-causal-vs-non-causal gene assignments across the 227 loci to build a 2x2 contingency table. Run stats::fisher.test on each table to obtain the odds ratio and 95% CI for the concurrent, nearest-gene, nearest-concurrent, and moderate-impact sentinel variant candidate sets. Write all four odds ratios to fisher_exact_odds_ratios.tsv and validate against the reported values: concurrent OR=29, nearest-gene OR=45 [31-67] (P=1.0e-107), nearest-concurrent OR=84, moderate-impact OR=169 (Supplementary Table S7, Figure 3).

## Related tools

- **ProGeM** (Generates the candidate causal gene output tables (bottom-up summary, top-down scores, concurrent, nearest genes) that are cross-tabulated against reference causal genes) — https://github.com/ds763/ProGeM
- **R** (Used with data.table to load output tables and stats::fisher.test to compute odds ratios and 95% confidence intervals per candidate definition)

## Examples

```
fisher.test(matrix(c(TP, FP, FN, TN), nrow=2))  # OR with 95% CI per candidate definition; write ORs to fisher_exact_odds_ratios.tsv
```

## Evaluation signals

- Recomputed nearest-gene odds ratio equals OR = 45 with 95% CI [31-67] and P = 1.0 x 10^-107, matching Supplementary Table S7 / Figure 3
- Recomputed concurrent, nearest-concurrent, and moderate-impact odds ratios equal 29, 84, and 169 respectively
- Each candidate definition yields a valid 2x2 contingency table whose margins sum to the gene/loci universe defined by the 227 curated mQTLs
- fisher_exact_odds_ratios.tsv contains exactly four rows (concurrent, nearest-gene, nearest-concurrent, moderate-impact) each with an OR and 95% CI
- Sensitivity check: ~98% of the 227 loci have the expert-curated gene among ProGeM candidates, and ~69% of causal candidates are nearest to the sentinel variant, consistent with reported results

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of the nearest genes, which inflates the nearest-gene odds ratio
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG used both by ProGeM and as a guide for the expert literature review), so observed sensitivity at mQTLs may be inflated
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex disease traits requires further research, dependent on availability of high-confidence reference datasets

## Evidence

- [full_text] For each candidate definition, construct a 2x2 contingency table crossing candidate-vs-non-candidate against reference-causal-vs-non-causal gene assignments across the 227 loci.: [evidence span withheld — anchor status: not_found]
- [results] Reported enrichment values to validate against: [evidence span withheld — anchor status: not_found]
- [readme] Candidate output file structure from README: "Each row provides annotation of a *"bottom-up"* candidate mediating gene, the corresponding sentinel rsID is indicated in **column 1**; **columns 2-4** provide basic annotation of candidate"
- [readme] Definition of concurrent candidates: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant. These genes are so-called "concurrent" genes"
- [abstract] Nearest-gene dominance finding: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [discussion] Reference dataset caveat: "the preparation of the reference causal gene assignments at the 227 mQTL, which could have been subject to bias, such as the prioritization of the nearest genes"
