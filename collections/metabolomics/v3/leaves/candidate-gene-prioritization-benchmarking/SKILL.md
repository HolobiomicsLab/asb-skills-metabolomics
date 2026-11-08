---
name: candidate-gene-prioritization-benchmarking
description: Use when validating a candidate causal gene prioritization tool or component
  against curated molecular QTL reference assignments, to quantify how many true positive
  causal genes are recovered and how many false positives are introduced.
when_to_use_negative:
- 'Input QTLs are trans-pQTLs: ProGeM''s top-down strategy is not directly applicable
  and would call for a different top-down strategy.'
- No high-confidence reference causal gene set is available for the QTL dataset, so
  sensitivity cannot be rigorously assessed.
- The goal is running ProGeM itself on new sentinel variants rather than evaluating
  its components against a curated reference.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Framework whose bottom-up and top-down component outputs are benchmarked for
    sensitivity and specificity against curated mQTL causal genes
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Environment for computing sensitivity/specificity from ProGeM summary files
    and the reference causal gene set
- name: data.table
  role: R package used with fread to load the ProGeM summary outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:e55bb6fe436b03126f3c88a9e26bfdcf81b0ce852d5376035d47b3ff2d5ae43a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - data.table
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# candidate-gene-prioritization-benchmarking

## Summary

Benchmark the sensitivity and specificity of the bottom-up and top-down components of ProGeM, applied individually, against a reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1), computing per-component recovery rates and precision and comparing them to reported benchmarks.

## When to use

Use when validating a candidate causal gene prioritization tool or component against curated molecular QTL reference assignments, to quantify how many true positive causal genes are recovered and how many false positives are introduced.

## When NOT to use

- Input QTLs are trans-pQTLs: ProGeM's top-down strategy is not directly applicable and would call for a different top-down strategy.
- No high-confidence reference causal gene set is available for the QTL dataset, so sensitivity cannot be rigorously assessed.
- The goal is running ProGeM itself on new sentinel variants rather than evaluating its components against a curated reference.

## Inputs

- OUTPUT_bottom_up_summary.txt (ProGeM bottom-up candidate table: LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, nearest/second_nearest/third_nearest/LD_overlapping/lead_eQTL/proxy_eQTL booleans, lead_IMPACT/proxy_IMPACT)
- OUTPUT_top_down_scores.txt (ProGeM top-down candidate table: LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, GO/KEGG/MGI/orphanet/reactome booleans, score out of 5)
- OUTPUT_cooccurring_candidates.txt (genes appearing as both bottom-up AND top-down candidates per sentinel variant)
- Reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1)

## Outputs

- benchmark_sensitivity_specificity.csv (per-component sensitivity, specificity and candidate counts out of 227 loci)

## How to apply

Load the three ProGeM summary outputs (OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt) into R with data.table::fread, and load the reference set of 227 curated metabolite QTL causal gene assignments from Supplementary Table S1, mapping each reference causal gene to its sentinel locus. For each component (bottom-up, top-down, co-occurring), compute sensitivity as the fraction of the 227 loci whose curated gene appears in the component list, and specificity as the proportion of candidate gene assignments that correspond to a reference causal gene. For co-occurring candidates, restrict to genes present in both the bottom-up and top-down lists for the same sentinel variant (expected 187/227, i.e. 82%). Compare computed values against reported benchmarks (bottom-up 95% sensitivity/0.384 specificity, top-down 85%/0.459, co-occurring 0.846), flag deviations, and write per-component results to benchmark_sensitivity_specificity.csv.

## Related tools

- **ProGeM** (Framework whose bottom-up and top-down component outputs are benchmarked for sensitivity and specificity against curated mQTL causal genes) — https://github.com/ds763/ProGeM
- **R** (Environment for computing sensitivity/specificity from ProGeM summary files and the reference causal gene set)
- **data.table** (R package used with fread to load the ProGeM summary outputs)

## Examples

```
bu <- data.table::fread('OUTPUT_bottom_up_summary.txt'); td <- data.table::fread('OUTPUT_top_down_scores.txt')
```

## Evaluation signals

- Bottom-up component recovers 216/227 (95%) of reference true positive causal genes
- Top-down component recovers 194/227 (85%) of reference true positive causal genes
- Co-occurring candidates recover 187/227 (~82%; reported specificity benchmark 0.846)
- Specificity values match reported benchmarks: bottom-up 0.384, top-down 0.459
- Per-component metrics and candidate counts are written to benchmark_sensitivity_specificity.csv without deviation flags

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of the nearest genes
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed sensitivity at mQTLs may be inflated
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex traits requires further research and depends on availability of high-confidence reference datasets

## Evidence

- [results] Applied individually to the 227 mQTLs, ProGeM's bottom-up component identified 216 (95%) and its top-down component identified 194 (85%) of the reference true positive causal genes.: [evidence span withheld — anchor status: not_found]
- [discussion] Reference set availability: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
- [readme] Co-occurring candidate definition: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant"
- [discussion] Sensitivity of ProGeM: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [readme] Top-down scoring: "column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator"
