---
name: sensitivity-false-positive-comparison-across-tools
description: Use when you have a curated reference set of QTLs with high-confidence
  causal genes and need to quantify and compare the sensitivity and false-positive
  burden of gene prioritization tools at those loci.
when_to_use_negative:
- No high-confidence reference causal gene set is available for the loci of interest
  — the article stresses that such datasets are a dependency for sensitivity assessment
- Evaluating trans-pQTL loci, where ProGeM's top-down strategy is not directly applicable
  and a different approach is needed
- The question is de novo gene prioritization rather than head-to-head tool benchmarking
  against known causal genes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: SNiPA
  role: Phenotype-agnostic baseline tool; its block annotation tool supplies candidate
    causal gene calls per sentinel locus for benchmarking
- name: ProGeM
  role: Tool under evaluation; bottom-up summary (OUTPUT_bottom_up_summary.txt) and
    combined candidate strategy provide the comparison arms
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Implementation language for ProGeM and for computing sensitivity/false-positive
    metrics and assembling the benchmark table
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:f8f976ee1084e32bd727631bdae20685ae4fbe108a5ef4afad6e7c6da944b71e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - SNiPA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sensitivity-false-positive-comparison-across-tools

## Summary

Benchmark candidate causal gene prioritization tools (e.g. SNiPA vs ProGeM) against a curated reference causal gene set, computing sensitivity as fraction of reference genes recovered and counting non-reference candidate genes as false positives.

## When to use

Use when you have a curated reference set of QTLs with high-confidence causal genes and need to quantify and compare the sensitivity and false-positive burden of gene prioritization tools at those loci.

## When NOT to use

- No high-confidence reference causal gene set is available for the loci of interest — the article stresses that such datasets are a dependency for sensitivity assessment
- Evaluating trans-pQTL loci, where ProGeM's top-down strategy is not directly applicable and a different approach is needed
- The question is de novo gene prioritization rather than head-to-head tool benchmarking against known causal genes

## Inputs

- Reference set of 227 curated metabolite QTLs with expert-assigned high-confidence causal genes (Supplementary Table S1)
- 218 recognizable sentinel variant rsIDs with GRCh37 coordinates
- SNiPA candidate causal gene calls per locus from the block annotation tool (GRCh37, 1000 Genomes Phase 3 v5, European, r2=0.8)
- ProGeM 'OUTPUT_bottom_up_summary.txt' candidate mediating genes mapped to sentinel rsIDs
- Combined ProGeM integrated candidate strategy output

## Outputs

- Per-tool sensitivity values over the 218 sentinels (SNiPA 92%, ProGeM bottom-up 96%, combined ProGeM 83%)
- Per-tool false-positive counts (SNiPA 1315, ProGeM bottom-up 827, combined ProGeM 207)
- Tab-separated comparison table snipa_vs_progem_bottomup_benchmark.tsv

## How to apply

Assemble a reference set (here 227 curated metabolite QTLs with 218 recognizable sentinel rsIDs) and a per-locus list of expert-curated true causal genes. Run each tool over the same sentinel variants — e.g. submit rsIDs to SNiPA's web 'block annotation' tool configured for GRCh37, 1000 Genomes Phase 3 v5, European population, r2=0.8 — and collect each tool's candidate genes per locus. For each tool, compute sensitivity as the fraction of the 218 reference causal genes recovered (SNiPA: 201/218 = 92%; ProGeM bottom-up: 209/218 = 96%; combined ProGeM components: 83%) and count candidate genes absent from the reference set as false positives (SNiPA: 1315; ProGeM bottom-up: 827; combined ProGeM: 207). Assemble all metrics into a single tab-separated comparison table. Validate by exact integer agreement with expected sensitivity ratios and false-positive counts.

## Related tools

- **SNiPA** (Phenotype-agnostic baseline tool; its block annotation tool supplies candidate causal gene calls per sentinel locus for benchmarking)
- **ProGeM** (Tool under evaluation; bottom-up summary (OUTPUT_bottom_up_summary.txt) and combined candidate strategy provide the comparison arms) — https://github.com/ds763/ProGeM
- **R** (Implementation language for ProGeM and for computing sensitivity/false-positive metrics and assembling the benchmark table) — https://github.com/ds763/ProGeM

## Examples

```
Rscript -e 'snipa_sens <- 201/218; progem_sens <- 209/218; write.table(benchmark, "snipa_vs_progem_bottomup_benchmark.tsv", sep="\t")'
```

## Evaluation signals

- SNiPA sensitivity reproduces the exact ratio 201/218 = 92%
- ProGeM bottom-up sensitivity reproduces 209/218 = 96% with 827 false positives
- Combined ProGeM components yield 83% sensitivity with 207 false positives
- SNiPA false-positive count is exactly 1315 (candidate genes not in the reference set)
- All metrics match expected benchmark values within exact integer agreement in snipa_vs_progem_bottomup_benchmark.tsv

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of nearest genes
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent, so observed sensitivity at mQTLs may be inflated
- KEGG was utilized both by ProGeM and as a guide for the expert literature review, further complicating independence of the benchmark

## Evidence

- [results] SNiPA benchmark sensitivity: [evidence span withheld — anchor status: not_found]
- [intro] ProGeM benchmarking rationale: "we have also determined the sensitivity and specificity of ProGeM using two molecular QTL datasets, comprising 227 mQTLs and 562 cis-pQTLs"
- [abstract] Reference dataset: "a reference set of 227 previously reported and extensively curated metabolite QTLs"
- [readme] ProGeM bottom-up candidate gene definition: "Each row provides annotation of a *"bottom-up"* candidate mediating gene, the corresponding sentinel rsID is indicated in **column 1**"
- [intro] SNiPA as comparator tool: "tools such as the Single Nucleotide Polymorphisms Annotator (SNiPA) and the Functional Mapping and Annotation of GWAS tool (FUMA) integrate various positional, regulatory and cis-eQTL datasets"
- [discussion] Reference dataset availability: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
