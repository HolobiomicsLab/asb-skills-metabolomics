---
name: gwas-locus-ld-block-annotation
description: Use when you need to annotate GWAS/mQTL sentinel loci with candidate
  causal genes via LD-block annotation, or when benchmarking phenotype-agnostic annotation
  tools like SNiPA against a reference causal gene set.
when_to_use_negative:
- The locus set is trans-pQTLs, since ProGeM's top-down strategy is not directly applicable
  there and would need a different approach
- No high-confidence reference causal gene set exists for your loci, since sensitivity/specificity
  cannot be computed without one
- You need trait-agnostic annotation only and have no interest in benchmarking against
  phenotype-informed prioritization
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: SNiPA
  role: Phenotype-agnostic web server used for 'block annotation' of sentinel loci
    to retrieve candidate causal genes (GRCh37, 1000 Genomes Phase 3 v5, European,
    r2=0.8)
- name: ProGeM
  role: Phenotype-informed R pipeline whose bottom-up and combined components are
    benchmarked against SNiPA calls; run with ProGeM_settings.R parameters and read
    from OUTPUT_bottom_up_summary.txt
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime for ProGeM and for computing sensitivity/false-positive metrics and
    assembling the benchmark table
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:7779823bc24bc77e155efe38d92bcfb77e7f372dbe6ae663cfb2bc6eb2cc62b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - SNiPA
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gwas-locus-ld-block-annotation

## Summary

Applies the SNiPA web server 'block annotation' tool to retrieve phenotype-agnostic candidate causal genes per GWAS/mQTL locus, and benchmarks its sensitivity and false-positive burden against the phenotype-informed ProGeM framework on a curated reference set of 227 mQTLs.

## When to use

Use when you need to annotate GWAS/mQTL sentinel loci with candidate causal genes via LD-block annotation, or when benchmarking phenotype-agnostic annotation tools like SNiPA against a reference causal gene set.

## When NOT to use

- The locus set is trans-pQTLs, since ProGeM's top-down strategy is not directly applicable there and would need a different approach
- No high-confidence reference causal gene set exists for your loci, since sensitivity/specificity cannot be computed without one
- You need trait-agnostic annotation only and have no interest in benchmarking against phenotype-informed prioritization

## Inputs

- Tab-separated .txt of sentinel variant rsIDs, chromosomes, and GRCh37 start/end coordinates (e.g. rs1801133 | 1 | 11856378 | 11856378)
- Tab-separated .txt of proxy variants with PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, and r2 columns
- VEP output .txt for sentinels and proxies with sentinel and proxy rsID columns
- Reference set of 227 curated metabolite QTLs with expert-assigned causal genes (Supplementary Table S1)
- ProGeM bottom-up summary file (OUTPUT_bottom_up_summary.txt)

## Outputs

- SNiPA candidate causal gene calls per locus from the block annotation tool
- Per-tool sensitivity and false-positive counts over the 218 recognizable sentinels
- Tab-separated benchmark table snipa_vs_progem_bottomup_benchmark.tsv

## How to apply

Submit the recognizable sentinel variant rsIDs (218 of the 227 curated mQTLs were recognizable) to the SNiPA 'block annotation' tool configured for GRCh37, 1000 Genomes Phase 3 v5, European population, and r2=0.8 to obtain SNiPA's candidate causal gene calls per locus. In parallel, run the ProGeM R pipeline (bottom-up component) using sentinel and proxy inputs and read candidate mediating genes from OUTPUT_bottom_up_summary.txt, mapping them to sentinel rsIDs. Compute sensitivity for each tool as the fraction of reference causal genes recovered (SNiPA: 201/218 = 92%; ProGeM bottom-up: 209/218 = 96%) and count candidates absent from the reference set as false positives (SNiPA: 1315; ProGeM bottom-up: 827; combined ProGeM strategy: 83% sensitivity with 207 false positives). Assemble the metrics into a tab-separated comparison table for validation against the expected integer counts.

## Related tools

- **SNiPA** (Phenotype-agnostic web server used for 'block annotation' of sentinel loci to retrieve candidate causal genes (GRCh37, 1000 Genomes Phase 3 v5, European, r2=0.8))
- **ProGeM** (Phenotype-informed R pipeline whose bottom-up and combined components are benchmarked against SNiPA calls; run with ProGeM_settings.R parameters and read from OUTPUT_bottom_up_summary.txt) — https://github.com/ds763/ProGeM
- **R** (Runtime for ProGeM and for computing sensitivity/false-positive metrics and assembling the benchmark table) — https://github.com/ds763/ProGeM

## Examples

```
Rscript ProGeM_commands.R  # after sourcing ProGeM_settings.R, read OUTPUT_bottom_up_summary.txt and compute sensitivity as n_reference_recovered/218
```

## Evaluation signals

- SNiPA sensitivity reproduces 201/218 = 92% of reference causal genes recovered with exact integer agreement
- SNiPA false-positive count equals 1315 candidate genes outside the reference set
- ProGeM bottom-up sensitivity reproduces 209/218 = 96% with 827 false positives
- Combined ProGeM components give 83% sensitivity with 207 false positives
- Output table snipa_vs_progem_bottomup_benchmark.tsv is valid tab-separated with matching metric values

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be biased, e.g. by prioritization of the nearest genes
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent, so observed mQTL sensitivity may be inflated
- KEGG was used both by ProGeM and as a guide for the expert literature review
- Whether genomic proximity is a good indicator of true causal genes for other molecular QTLs requires further research

## Evidence

- [results] SNiPA achieved 92% sensitivity (201/218 true positives) with 1315 false positives: [evidence span withheld — anchor status: not_found]
- [methods] Proxies and r2 threshold: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [intro] Sensitivity of existing tools not rigorously assessed: [evidence span withheld — anchor status: not_found]
- [discussion] ProGeM missed only 4 of 227 reference genes: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [readme] Bottom-up summary file columns: "Each row provides annotation of a *"bottom-up"* candidate mediating gene, the corresponding sentinel rsID is indicated in **column 1**"
