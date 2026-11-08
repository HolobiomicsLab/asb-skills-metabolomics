---
name: sensitivity-specificity-computation
description: Use when you have ProGeM summary outputs (OUTPUT_bottom_up_summary.txt,
  OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt) and a curated reference
  causal gene set, and need to quantify how well each candidate component recovers
  known causal genes.
when_to_use_negative:
- The QTL dataset lacks a high-confidence reference set of curated causal gene assignments,
  so sensitivity/specificity cannot be computed against known true positives.
- The goal is prioritizing candidates at trans-pQTLs, where ProGeM's top-down strategy
  is not directly applicable and would need a different approach.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: ProGeM
  role: Produces the bottom-up, top-down, and co-occurring candidate gene summary
    files that are benchmarked for sensitivity and specificity
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Environment in which the benchmark sensitivity/specificity computation over
    the ProGeM output tables is performed
- name: data.table
  role: R package used to fread the three ProGeM summary output files into data tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:eaf7faa727da189eb8b1b355d519fcfa83730a85f90e17a986c0e128c5f4882e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - data.table
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sensitivity-specificity-computation

## Summary

Benchmark the sensitivity and specificity of ProGeM's bottom-up, top-down, and co-occurring candidate gene components against the reference set of 227 curated metabolite QTL causal gene assignments, reporting per-component recovery rates and true-positive proportions.

## When to use

Use when you have ProGeM summary outputs (OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt) and a curated reference causal gene set, and need to quantify how well each candidate component recovers known causal genes.

## When NOT to use

- The QTL dataset lacks a high-confidence reference set of curated causal gene assignments, so sensitivity/specificity cannot be computed against known true positives.
- The goal is prioritizing candidates at trans-pQTLs, where ProGeM's top-down strategy is not directly applicable and would need a different approach.

## Inputs

- OUTPUT_bottom_up_summary.txt (tab-separated ProGeM bottom-up candidate table with LEAD_rsID, ensembl_id, hgnc_symbol, nearest/LD/eQTL/IMPACT boolean columns)
- OUTPUT_top_down_scores.txt (tab-separated ProGeM top-down table with GO/KEGG/MGI/orphanet/reactome booleans and score out of 5)
- OUTPUT_cooccurring_candidates.txt (ProGeM co-occurring/concurrent candidate list)
- Reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1)

## Outputs

- benchmark_sensitivity_specificity.csv with per-component sensitivity, specificity and candidate counts
- Per-component recovery counts out of 227 loci (expected: bottom-up 216, top-down 194, co-occurring 187)

## How to apply

Load the three ProGeM summary outputs into R with data.table::fread and load the reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1), mapping each reference causal gene to its sentinel locus. For each component, compute sensitivity as the fraction of the 227 loci whose curated gene appears in the component list, and specificity as the proportion of candidate gene assignments that correspond to a reference causal gene. For co-occurring candidates, restrict to genes present in BOTH the bottom-up and top-down lists for the same sentinel variant (ProGeM's 'concurrent' genes). Compare computed values against the reported benchmarks: bottom-up 216/227 (95%) sensitivity with 0.384 specificity, top-down 194/227 (85%) with 0.459 specificity, and co-occurring 187/227 (~82%, reported as 0.846). Write per-component sensitivity, specificity and candidate counts to benchmark_sensitivity_specificity.csv and flag any deviation from the expected values.

## Related tools

- **ProGeM** (Produces the bottom-up, top-down, and co-occurring candidate gene summary files that are benchmarked for sensitivity and specificity) — https://github.com/ds763/ProGeM
- **R** (Environment in which the benchmark sensitivity/specificity computation over the ProGeM output tables is performed)
- **data.table** (R package used to fread the three ProGeM summary output files into data tables)

## Examples

```
bu <- data.table::fread('OUTPUT_bottom_up_summary.txt'); td <- data.table::fread('OUTPUT_top_down_scores.txt'); sens <- sum(ref$causal_gene %in% bu$hgnc_symbol[match(ref$LEAD_rsID, bu$LEAD_rsID)]) / 227
```

## Evaluation signals

- Bottom-up component recovers 216/227 (95%) of reference causal genes; deviations from this count flag a problem.
- Top-down component recovers 194/227 (85%) of reference causal genes with specificity 0.459.
- Co-occurring candidates restricted to genes in both bottom-up and top-down lists for the same sentinel number 187/227 (~82%, reported 0.846).
- Specificity for the bottom-up component is 0.384 (roughly 3 false positives per true positive, consistent with the paper's cis-eQTL finding).
- Every one of the 227 loci maps to exactly one reference causal gene before computing recovery fractions.

## Limitations

- Reference causal gene assignments at the 227 mQTLs may be biased, e.g. by prioritization of nearest genes.
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert review), so observed mQTL sensitivity may be inflated.
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex traits requires further research and depends on availability of high-confidence reference datasets.

## Evidence

- [other] bottom-up 95% sensitivity: [evidence span withheld — anchor status: not_found]
- [discussion] overall ProGeM sensitivity: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [readme] co-occurring candidate definition: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant.  These genes are so-called "concurrent" genes"
- [abstract] cis-eQTL false positive rate: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [discussion] reference dataset availability: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
