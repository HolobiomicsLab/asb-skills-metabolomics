---
name: sentinel-variant-gene-prioritization-benchmarking
description: Use when you have a curated reference set of causal gene assignments
  at QTL loci (e.g. 227 mQTLs) and need to quantify sensitivity and false-positive
  rates of candidate causal gene prioritization tools.
when_to_use_negative:
- Loci are trans-QTLs — ProGeM's top-down strategy is not directly applicable and
  'the prioritization of candidate causal genes at trans-pQTLs would call for a different
  top-down strategy'
- No high-confidence reference causal gene set exists for the trait, making sensitivity/false-positive
  computation impossible
- The goal is novel causal gene discovery rather than evaluating already-published
  prioritization tools against a reference
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SNiPA
  role: Phenotype-agnostic 'block annotation' tool benchmarked for sensitivity and
    false positives against the 227 mQTL reference set
- name: ProGeM
  role: R-based candidate causal gene prioritization framework whose bottom-up and
    combined components are the tools-under-test and comparators
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Execution environment for the ProGeM script, requiring GenomicAlignments,
    GenomicFeatures, GenomicRanges, biomaRt, and data.table
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:b63234f923638cc870e96226d9e2466305f98ad07918cd7e952b318d5d7b65ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - SNiPA
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sentinel-variant-gene-prioritization-benchmarking

## Summary

Benchmark phenotype-agnostic gene-prioritization tools (SNiPA) against ProGeM's bottom-up and combined candidate strategies by computing sensitivity and false-positive counts over the 227 curated metabolite QTL (mQTL) reference set with 218 recognizable sentinel rsIDs.

## When to use

Use when you have a curated reference set of causal gene assignments at QTL loci (e.g. 227 mQTLs) and need to quantify sensitivity and false-positive rates of candidate causal gene prioritization tools.

## When NOT to use

- Loci are trans-QTLs — ProGeM's top-down strategy is not directly applicable and 'the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy'
- No high-confidence reference causal gene set exists for the trait, making sensitivity/false-positive computation impossible
- The goal is novel causal gene discovery rather than evaluating already-published prioritization tools against a reference

## Inputs

- Tab-separated sentinel variant file with rsID, CHR, START, END columns (GRCh37 coordinates)
- Tab-separated proxy variant file with PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2 columns
- Variant Effect Predictor (VEP) output .txt for sentinels and proxies with IMPACT annotations
- ProGeM bottom-up summary output ('OUTPUT_bottom_up_summary.txt')
- Reference set of 227 curated metabolite QTLs with expert-assigned causal genes (218 recognizable sentinels)

## Outputs

- SNiPA sensitivity and false-positive counts per locus (target: 201/218 = 92% sensitivity, 1315 false positives)
- ProGeM bottom-up sensitivity and false positives (target: 209/218 = 96%, 827 false positives)
- Combined ProGeM performance (target: 83% sensitivity, 207 false positives)
- Tab-separated benchmark table saved as snipa_vs_progem_bottomup_benchmark.tsv

## How to apply

Submit the 218 recognizable sentinel rsIDs to the SNiPA web server 'block annotation' tool configured for GRCh37, 1000 Genomes Phase 3 v5, European population, and r2=0.8 to retrieve per-locus candidate causal gene calls. Run ProGeM's bottom-up pipeline (nearest 3 protein-coding genes, LD-region overlapping genes with 5 kb overhang, GTEx v6p cis-eQTL targets) to produce 'OUTPUT_bottom_up_summary.txt', then map candidate mediating genes back to sentinel rsIDs. Compute sensitivity as the fraction of the 218 reference causal genes recovered and count candidate genes absent from the reference set as false positives. Assemble SNiPA, ProGeM bottom-up, and combined ProGeM metrics into a tab-separated table (snipa_vs_progem_bottomup_benchmark.tsv) with exact integer agreement on expected counts.

## Related tools

- **SNiPA** (Phenotype-agnostic 'block annotation' tool benchmarked for sensitivity and false positives against the 227 mQTL reference set)
- **ProGeM** (R-based candidate causal gene prioritization framework whose bottom-up and combined components are the tools-under-test and comparators) — https://github.com/ds763/ProGeM
- **R** (Execution environment for the ProGeM script, requiring GenomicAlignments, GenomicFeatures, GenomicRanges, biomaRt, and data.table) — https://github.com/ds763/ProGeM

## Examples

```
Rscript -e "source('ProGeM_settings.R'); source('ProGeM_commands.R')"
```

## Evaluation signals

- SNiPA sensitivity exactly reproduces 201/218 (92%) and false positives equal 1315
- ProGeM bottom-up sensitivity exactly reproduces 209/218 (96%) with 827 false positives
- Combined ProGeM metrics give 83% sensitivity with 207 false positives
- Every row of snipa_vs_progem_bottomup_benchmark.tsv matches expected sensitivity ratios and FP counts in exact integer agreement
- ProGeM settings used default filters: protein-coding biotype, 3 nearest genes, r2=0.8 proxy threshold, +/-500 kb local window

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of nearest genes
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent, so observed sensitivity at mQTLs may be inflated
- KEGG was used both by ProGeM and as a guide for the expert literature review, confounding independence

## Evidence

- [abstract] For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [intro] the sensitivity of these tools, or the extent to which they are able to highlight 'true positive' causal genes, has not yet been rigorously assessed: [evidence span withheld — anchor status: not_found]
- [methods] ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8): "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [discussion] increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [readme] ProGeM will identify bottom-up candidate causal genes in two broad ways: "ProGeM will identify bottom-up candidate causal genes in two broad ways"
