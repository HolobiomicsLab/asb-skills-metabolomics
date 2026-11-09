---
name: fisher-exact-enrichment-testing
description: Use when you have categorical candidate-vs-reference gene assignments
  across curated loci and need odds ratios with 95% CIs to rank which annotation criteria
  are enriched for true positives.
when_to_use_negative:
- Candidate gene assignments have not yet been generated (no ProGeM OUTPUT_* tables)
  or no reference causal gene set is available to cross against
- The analysis targets trans-pQTLs, where ProGeM's top-down candidate strategy and
  this enrichment benchmarking design do not directly apply
- The goal is continuous-score evaluation rather than categorical candidate-vs-non-candidate
  enrichment; a 2x2 Fisher test is not appropriate
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: R
  role: stats::fisher.test computes odds ratios and 95% CIs on 2x2 contingency tables;
    data.table loads the ProGeM output tables
  repo: https://github.com/ds763/ProGeM
- name: ProGeM
  role: generates the candidate causal gene sets (bottom-up, top-down, cooccurring,
    nearest genes) whose enrichment for reference causal genes is tested
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:d7d0527352890b7193faa9c549ba32bfd5a28b146d91849694f1a22ada1dcf7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fisher-exact-enrichment-testing

## Summary

Quantify enrichment of true positive causal genes among candidate causal gene sets using 2x2 contingency tables and Fisher's exact test, as done in ProGeM's benchmarking of bottom-up annotation criteria at 227 curated metabolite QTLs (e.g. nearest-gene set OR=45, moderate-impact set OR=169).

## When to use

Use when you have categorical candidate-vs-reference gene assignments across curated loci and need odds ratios with 95% CIs to rank which annotation criteria are enriched for true positives.

## When NOT to use

- Candidate gene assignments have not yet been generated (no ProGeM OUTPUT_* tables) or no reference causal gene set is available to cross against
- The analysis targets trans-pQTLs, where ProGeM's top-down candidate strategy and this enrichment benchmarking design do not directly apply
- The goal is continuous-score evaluation rather than categorical candidate-vs-non-candidate enrichment; a 2x2 Fisher test is not appropriate

## Inputs

- ProGeM candidate gene output tables: OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt, OUTPUT_nearest_genes.txt
- Reference causal gene assignments for the 227 curated metabolite QTLs (Supplementary Table S1 mQTL reference dataset)

## Outputs

- fisher_exact_odds_ratios.tsv containing odds ratios and 95% confidence intervals for the concurrent, nearest-gene, nearest-concurrent, and moderate-impact candidate sets
- Per-criterion Fisher's exact test odds ratios comparable to Supplementary Table S7 / Figure 3 values

## How to apply

Load the ProGeM candidate gene output tables (e.g. 'OUTPUT_bottom_up_summary.txt', 'OUTPUT_top_down_scores.txt') using data.table in R, together with the reference causal gene assignments for the 227 curated mQTLs (Supplementary Table S1). For each candidate definition (concurrent, nearest-gene, nearest-concurrent, moderate-impact sentinel variant), construct a 2x2 contingency table crossing candidate-vs-non-candidate against reference-causal-vs-non-causal gene assignments across the 227 loci. Run stats::fisher.test on each table to obtain the odds ratio and its 95% confidence interval. Write all four odds ratios with CIs to fisher_exact_odds_ratios.tsv and verify they reproduce the reported enrichment values (concurrent OR=29, nearest-gene OR=45, nearest-concurrent OR=84, moderate-impact OR=169) tabulated in Supplementary Table S7 and plotted in Figure 3.

## Related tools

- **R** (stats::fisher.test computes odds ratios and 95% CIs on 2x2 contingency tables; data.table loads the ProGeM output tables) — https://github.com/ds763/ProGeM
- **ProGeM** (generates the candidate causal gene sets (bottom-up, top-down, cooccurring, nearest genes) whose enrichment for reference causal genes is tested) — https://github.com/ds763/ProGeM

## Examples

```
Rscript -e 'library(data.table); x<-fread("OUTPUT_nearest_genes.txt"); ct<-table(candidate=x$candidate, causal=x$ref_causal); fisher.test(ct)'
```

## Evaluation signals

- Recomputed odds ratios match the reported values: concurrent OR=29, nearest-gene set OR=45 [31-67] with P=1.0e-107, nearest-concurrent OR=84, moderate-impact OR=169
- All four candidate definitions yield ORs and 95% CIs that reproduce Supplementary Table S7 and Figure 3
- 2x2 contingency tables are built over exactly the 227 curated mQTL loci with candidate-vs-non-candidate crossed against reference-causal-vs-non-causal
- Output file fisher_exact_odds_ratios.tsv contains odds ratios with confidence intervals for all four candidate sets

## Limitations

- The reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of nearest genes, which inflates the proximity enrichment
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed enrichment at mQTLs may be inflated
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex traits requires further research and depends on availability of high-confidence reference datasets

## Evidence

- [abstract] 69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [discussion] reference causal gene assignments could be biased toward nearest genes: "the preparation of the reference causal gene assignments at the 227 mQTL, which could have been subject to bias, such as the prioritization of the nearest genes"
- [discussion] sensitivity at mQTLs may be inflated due to KEGG overlap: "the KEGG database was utilized both by ProGeM and as a guide for the expert literature review"
- [discussion] top-down trans-pQTL caveat: "the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy to the one adopted here"
- [readme] bottom-up summary annotates nearest genes: "ProGeM will identify the nearest genes to each sentinel variant.  The user can specify how many nearest genes they would like to pull out (default is 3)"
