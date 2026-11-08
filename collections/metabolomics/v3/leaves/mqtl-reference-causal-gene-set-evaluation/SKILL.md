---
name: mqtl-reference-causal-gene-set-evaluation
description: Use when you have a curated molecular QTL reference set with known causal
  genes and need to quantify how many true positives a prioritization tool recovers
  and how many false positives it introduces.
when_to_use_negative:
- Your QTLs are trans-pQTLs — the article notes ProGeM's top-down strategy is 'not
  directly applicable to trans-pQTLs and would need a different approach'
- You lack a high-confidence reference causal gene set — the article states generalizing
  sensitivity conclusions 'will depend on the availability of high-confidence reference
  datasets'
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SNiPA
  role: Baseline phenotype-agnostic prioritization tool; candidate causal genes retrieved
    via its web 'block annotation' tool for benchmarking against the reference set
- name: ProGeM
  role: Trait-focused prioritization framework whose bottom-up and combined candidate
    outputs are benchmarked; configured via ProGeM_settings.R and run as an R script
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime for ProGeM scripts (ProGeM_settings.R, ProGeM_functions.R, ProGeM_commands.R)
    and for computing sensitivity/false-positive metrics over the benchmark table
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:4bc38677b6a7633ce8d1c78026ce51fd2e889d02c6d5484cde021336bed8fdc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - SNiPA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mqtl-reference-causal-gene-set-evaluation

## Summary

Benchmark the sensitivity and false-positive rate of causal gene prioritization tools (SNiPA, ProGeM bottom-up and combined) against a curated reference set of 227 metabolite QTLs with 218 recognizable sentinels and known causal genes.

## When to use

Use when you have a curated molecular QTL reference set with known causal genes and need to quantify how many true positives a prioritization tool recovers and how many false positives it introduces.

## When NOT to use

- Your QTLs are trans-pQTLs — the article notes ProGeM's top-down strategy is 'not directly applicable to trans-pQTLs and would need a different approach'
- You lack a high-confidence reference causal gene set — the article states generalizing sensitivity conclusions 'will depend on the availability of high-confidence reference datasets'

## Inputs

- Reference set of 227 curated metabolite QTLs with expert-assigned causal genes (Supplementary Table S1)
- 218 recognizable sentinel variant rsIDs with GRCh37 coordinates (tab-separated txt: rsID | CHR | START | END)
- Proxy variant table with sentinel mapping and r2 values (tab-separated txt)
- ProGeM bottom-up summary ('OUTPUT_bottom_up_summary.txt') mapping candidate mediating genes to sentinel rsIDs
- SNiPA block annotation output (GRCh37, 1000G Phase 3 v5, European, r2=0.8)

## Outputs

- Sensitivity and false-positive counts per tool (SNiPA, ProGeM bottom-up, combined ProGeM)
- Tab-separated benchmark table: snipa_vs_progem_bottomup_benchmark.tsv

## How to apply

Submit the 218 recognizable sentinel rsIDs to SNiPA's web 'block annotation' tool configured for GRCh37, 1000 Genomes Phase 3 v5, European population, and r2=0.8, then record its candidate causal gene calls per locus. In parallel, run ProGeM's bottom-up component (nearest genes, LD-region overlapping genes, and GTEx cis-eQTL targets for sentinels/proxies, filtered on protein-coding biotype) and collect its candidates from 'OUTPUT_bottom_up_summary.txt'. Compute sensitivity as the fraction of the 218 reference causal genes recovered, and false positives as candidate genes absent from the reference set, per tool and for ProGeM's combined (cooccurring candidate) strategy. Expected benchmarks: SNiPA 201/218 = 92% sensitivity with 1315 false positives; ProGeM bottom-up 209/218 = 96% with 827 false positives; combined ProGeM 83% sensitivity with 207 false positives. Assemble the metrics into a tab-separated table (snipa_vs_progem_bottomup_benchmark.tsv) and validate exact integer agreement with these ratios.

## Related tools

- **SNiPA** (Baseline phenotype-agnostic prioritization tool; candidate causal genes retrieved via its web 'block annotation' tool for benchmarking against the reference set)
- **ProGeM** (Trait-focused prioritization framework whose bottom-up and combined candidate outputs are benchmarked; configured via ProGeM_settings.R and run as an R script) — https://github.com/ds763/ProGeM
- **R** (Runtime for ProGeM scripts (ProGeM_settings.R, ProGeM_functions.R, ProGeM_commands.R) and for computing sensitivity/false-positive metrics over the benchmark table) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R")
```

## Evaluation signals

- SNiPA sensitivity must reproduce 201/218 = 92% and yield exactly 1315 false positives
- ProGeM bottom-up sensitivity must reproduce 209/218 = 96% with exactly 827 false positives
- Combined ProGeM components must yield 83% sensitivity with 207 false positives
- Every sentinel sent to SNiPA block annotation must have GRCh37/1000G-compatible rsIDs (218 recognizable of 227 curated mQTLs)
- Final benchmark TSV must match expected counts in exact integer agreement with no missing loci

## Limitations

- Reference causal gene assignments could be biased, e.g. 'the prioritization of the nearest genes'
- ProGeM and the curated mQTL dataset are not independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so mQTL sensitivity 'may be inflated'
- cis-eQTL data are noisy: 'cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment'

## Evidence

- [intro] the sensitivity of these tools, or the extent to which they are able to highlight 'true positive' causal genes, has not yet been rigorously assessed: [evidence span withheld — anchor status: not_found]
- [abstract] For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [discussion] increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [methods] A tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of your sentinel variants: "A tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of your sentinel variants"
- [readme] ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant"
