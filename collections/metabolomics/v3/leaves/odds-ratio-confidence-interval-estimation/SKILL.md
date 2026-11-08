---
name: odds-ratio-confidence-interval-estimation
description: Use when you have binary candidate-vs-non-candidate gene assignments
  from a prioritization tool and a reference set of curated true causal genes, and
  you need to rank annotation criteria by enrichment odds ratio and P-value.
when_to_use_negative:
- You lack a high-confidence reference set of curated true causal gene assignments
  — enrichment against an unrepresentative reference set cannot be interpreted (the
  article notes sensitivity may be inflated because ProGeM and the mQTL reference
  dataset are not independent).
- Your QTLs are trans-pQTLs or other molecular QTLs where the mQTL-specific reference
  dataset and ProGeM top-down strategy do not apply.
- You need enrichment of continuous or ranked scores rather than binary candidate/non-candidate
  gene classifications — the 2x2 Fisher's exact contingency approach requires dichotomized
  membership.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for constructing the 2x2 contingency tables and running stats::fisher.test
    to obtain odds ratios, 95% confidence intervals, and P-values; data.table is used
    to load the ProGeM output tables
- name: ProGeM
  role: Produces the candidate causal gene annotation tables (bottom-up, top-down,
    cooccurring/concurrent, nearest genes) at the 227 mQTLs that are tested for enrichment
    against the reference causal gene set
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:afcfa833704dbcbe40c6cfffd3cb8865a2800e225244b9ada1cb4eb4a4b3aab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# odds-ratio-confidence-interval-estimation

## Summary

Quantify enrichment of true positive causal genes among ProGeM candidate gene annotation criteria at the 227 curated mQTLs by constructing 2x2 contingency tables and running Fisher's exact test in R, yielding odds ratios with 95% confidence intervals (e.g. nearest-gene set OR=45 [31-67]).

## When to use

Use when you have binary candidate-vs-non-candidate gene assignments from a prioritization tool and a reference set of curated true causal genes, and you need to rank annotation criteria by enrichment odds ratio and P-value.

## When NOT to use

- You lack a high-confidence reference set of curated true causal gene assignments — enrichment against an unrepresentative reference set cannot be interpreted (the article notes sensitivity may be inflated because ProGeM and the mQTL reference dataset are not independent).
- Your QTLs are trans-pQTLs or other molecular QTLs where the mQTL-specific reference dataset and ProGeM top-down strategy do not apply.
- You need enrichment of continuous or ranked scores rather than binary candidate/non-candidate gene classifications — the 2x2 Fisher's exact contingency approach requires dichotomized membership.

## Inputs

- OUTPUT_bottom_up_summary.txt (ProGeM bottom-up candidate table with nearest/second_nearest/third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT boolean columns)
- OUTPUT_top_down_scores.txt (ProGeM top-down metabolic-gene scoring table, score out of 5 databases)
- OUTPUT_cooccurring_candidates.txt (concurrent bottom-up AND top-down candidates)
- OUTPUT_nearest_genes.txt (genes nearest to each sentinel variant)
- Supplementary Table S1 mQTL reference dataset (reference causal gene assignments for the 227 curated metabolite QTLs)

## Outputs

- fisher_exact_odds_ratios.tsv (odds ratios with 95% confidence intervals and P-values for concurrent, nearest-gene, nearest-concurrent, and moderate-impact candidate sets)
- Per-criterion 2x2 contingency tables of candidate-vs-non-candidate against reference-causal-vs-non-causal gene assignments

## How to apply

Load the four ProGeM candidate output tables ('OUTPUT_bottom_up_summary.txt', 'OUTPUT_top_down_scores.txt', 'OUTPUT_cooccurring_candidates.txt', 'OUTPUT_nearest_genes.txt') with data.table in R, plus the reference causal gene assignments for the 227 curated metabolite QTLs (Supplementary Table S1 mQTL reference dataset). For each candidate definition (concurrent, nearest-gene, nearest-concurrent, and moderate-impact sentinel variant candidate sets), build a 2x2 contingency table crossing candidate-vs-non-candidate against reference-causal-vs-non-causal assignments across the 227 loci. Run stats::fisher.test on each table to extract the odds ratio, 95% confidence interval, and P-value. Write all four odds ratios with confidence intervals to fisher_exact_odds_ratios.tsv, then validate against the reported enrichment values: concurrent OR=29, nearest-gene OR=45 [31-67] (P = 1.0e-107), nearest-concurrent OR=84, moderate-impact OR=169 as tabulated in Supplementary Table S7 and plotted in Figure 3.

## Related tools

- **R** (Environment for constructing the 2x2 contingency tables and running stats::fisher.test to obtain odds ratios, 95% confidence intervals, and P-values; data.table is used to load the ProGeM output tables)
- **ProGeM** (Produces the candidate causal gene annotation tables (bottom-up, top-down, cooccurring/concurrent, nearest genes) at the 227 mQTLs that are tested for enrichment against the reference causal gene set) — https://github.com/ds763/ProGeM

## Examples

```
fisher.test(matrix(c(n11, n12, n21, n22), nrow=2))$estimate
```

## Evaluation signals

- Recomputed odds ratios match the reported values exactly: concurrent OR=29, nearest-gene OR=45, nearest-concurrent OR=84, moderate-impact sentinel variant OR=169.
- Nearest-gene enrichment reproduces OR = 45 [31-67], P = 1.0 x 10^-107, matching Supplementary Table S7 and Figure 3.
- Each contingency table's margins are consistent with the 227 curated mQTL loci and the reference causal gene assignments in Supplementary Table S1.
- All four odds ratios carry 95% confidence intervals and the output table contains exactly one row per annotation criterion (concurrent, nearest-gene, nearest-concurrent, moderate-impact).

## Limitations

- The preparation of the reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as the prioritization of the nearest genes, which would inflate the nearest-gene enrichment signal.
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed sensitivity/enrichment at mQTLs may be inflated.
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex disease traits requires further research, and generalization depends on availability of high-confidence reference datasets.

## Evidence

- [methods] Fisher's exact enrichment of annotation criteria on the mQTL dataset: [evidence span withheld — anchor status: not_found]
- [supplementary] Nearest-gene set enrichment value: [evidence span withheld — anchor status: not_found]
- [abstract] Genomic proximity is the most reliable indicator of true positives: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [readme] Candidate definitions come from the ProGeM bottom-up output columns: "ProGeM will identify the nearest genes to each sentinel variant.  The user can specify how many nearest genes they would like to pull out (default is 3)"
- [abstract] Reference set of 227 curated mQTLs used for benchmarking: "a reference set of 227 previously reported and extensively curated metabolite QTLs"
