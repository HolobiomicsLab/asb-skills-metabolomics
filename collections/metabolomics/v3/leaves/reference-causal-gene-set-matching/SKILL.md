---
name: reference-causal-gene-set-matching
description: Use when you have prioritized candidate causal gene lists per sentinel
  variant and a curated reference set of true positive causal genes, and need per-component
  sensitivity (fraction of reference loci recovered) and specificity (true positives
  per candidate assignment).
when_to_use_negative:
- No curated reference causal gene set exists for your QTL dataset — sensitivity cannot
  be computed without high-confidence true positive assignments
- Evaluating trans-pQTL causal gene prioritization, where the article states ProGeM's
  top-down strategy is not directly applicable and 'would call for a different top-down
  strategy'
- Inputs are raw QTL summary statistics or sentinel variant lists rather than prioritized
  candidate gene lists per locus
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Source of the bottom-up, top-down and co-occurring candidate gene summary
    files being benchmarked
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Scripting environment for loading tables, computing sensitivity/specificity
    and writing the benchmark CSV
- name: data.table
  role: R package used via data.table::fread to load the summary outputs and reference
    table
  repo: https://cran.r-project.org/web/packages/data.table/data.table.pdf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:e7fc8728487d41141ba25ef56d2380b35cae094e6cab450b5577ffca11cdbf12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - data.table
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-causal-gene-set-matching

## Summary

Benchmark the sensitivity and specificity of causal gene prioritization outputs (ProGeM bottom-up, top-down, and co-occurring candidates) by matching them against a curated reference set of 227 metabolite QTL causal gene assignments.

## When to use

Use when you have prioritized candidate causal gene lists per sentinel variant and a curated reference set of true positive causal genes, and need per-component sensitivity (fraction of reference loci recovered) and specificity (true positives per candidate assignment).

## When NOT to use

- No curated reference causal gene set exists for your QTL dataset — sensitivity cannot be computed without high-confidence true positive assignments
- Evaluating trans-pQTL causal gene prioritization, where the article states ProGeM's top-down strategy is not directly applicable and 'would call for a different top-down strategy'
- Inputs are raw QTL summary statistics or sentinel variant lists rather than prioritized candidate gene lists per locus

## Inputs

- OUTPUT_bottom_up_summary.txt (ProGeM bottom-up candidate table with LEAD_rsID, ensembl_id, hgnc_symbol, boolean annotations)
- OUTPUT_top_down_scores.txt (ProGeM top-down candidate table with GO/KEGG/MGI/orphanet/reactome columns and database score out of 5)
- OUTPUT_cooccurring_candidates.txt (ProGeM co-occurring 'concurrent' gene candidates)
- Supplementary Table S1: reference set of 227 curated metabolite QTL causal gene assignments

## Outputs

- benchmark_sensitivity_specificity.csv with per-component sensitivity, specificity and candidate counts
- Per-component recovery counts (e.g. bottom-up 216/227, top-down 194/227, co-occurring 187/227)

## How to apply

Load the ProGeM outputs (OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt) into R with data.table::fread, plus the reference set of 227 curated metabolite QTL causal gene assignments from Supplementary Table S1, mapping each reference causal gene to its sentinel locus. For each candidate component compute sensitivity as the fraction of the 227 loci whose curated gene appears in the component list, and specificity as the proportion of candidate gene assignments that correspond to a reference causal gene. For co-occurring candidates, restrict to genes present in both the bottom-up and top-down lists for the same sentinel variant and report the count out of 227 (expected 187/227, i.e. 82%, or 0.846). Compare computed values against the published benchmarks (bottom-up 95% sensitivity / 0.384 specificity; top-down 85% sensitivity / 0.459 specificity) and flag any deviation. Write per-component sensitivity, specificity and candidate counts to benchmark_sensitivity_specificity.csv.

## Related tools

- **ProGeM** (Source of the bottom-up, top-down and co-occurring candidate gene summary files being benchmarked) — https://github.com/ds763/ProGeM
- **R** (Scripting environment for loading tables, computing sensitivity/specificity and writing the benchmark CSV)
- **data.table** (R package used via data.table::fread to load the summary outputs and reference table) — https://cran.r-project.org/web/packages/data.table/data.table.pdf

## Examples

```
bu <- data.table::fread('OUTPUT_bottom_up_summary.txt'); ref <- data.table::fread('SuppTableS1.txt'); sens <- mean(ref$causal_gene %in% bu$hgnc_symbol[match(ref$LEAD_rsID, bu$LEAD_rsID)])
```

## Evaluation signals

- Bottom-up sensitivity should be ~95% (216/227 reference causal genes recovered)
- Top-down sensitivity should be ~85% (194/227 recovered) with specificity ~0.459
- Co-occurring candidates should recover 187/227 loci (~82%, reported as 0.846)
- Overall union: the expert-curated gene was among ProGeM candidates for 98% of the 227 loci (only 4 of 227 missed)
- Every reference causal gene in Supplementary Table S1 must map to exactly one sentinel locus before counting

## Limitations

- Reference causal gene assignments could be biased, e.g. by prioritization of the nearest genes, inflating apparent sensitivity
- ProGeM and the curated mQTL dataset are not fully independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed mQTL sensitivity may be inflated
- Candidate assignments can outnumber true positives heavily: cis-eQTL data led to three false positive assignments for every one true positive, and bottom-up specificity is only 0.384

## Evidence

- [results] Applied individually to the 227 mQTLs, ProGeM's bottom-up component identified 216 (95%) and its top-down component identified 194 (85%) of the reference true positive causal genes.: [evidence span withheld — anchor status: not_found]
- [discussion] We have made our mQTL reference dataset available to the research community in Supplementary Table S1: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
- [readme] ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant"
- [discussion] the preparation of the reference causal gene assignments at the 227 mQTL, which could have been subject to bias, such as the prioritization of the nearest genes: "the preparation of the reference causal gene assignments at the 227 mQTL, which could have been subject to bias, such as the prioritization of the nearest genes"
- [abstract] cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
