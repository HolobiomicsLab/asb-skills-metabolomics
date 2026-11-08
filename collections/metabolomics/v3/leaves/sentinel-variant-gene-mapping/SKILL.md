---
name: sentinel-variant-gene-mapping
description: Use when you have sentinel variants from molecular QTL mapping (e.g.
  mQTLs or cis-pQTLs) with GRCh37 coordinates and need to prioritize which local genes
  are causal, using positional, cis-eQTL (GTEx) and metabolic annotation evidence.
when_to_use_negative:
- The goal is prioritizing causal genes at trans-pQTLs — the article states the top-down
  strategy adopted here is not directly applicable and 'would call for a different
  top-down strategy'
- The phenotype is not metabolic-related and no phenotype-specific .RData gene list
  has been prepared — the supplied annotated_metabolic_genes.RData covers metabolic-related
  phenotypes only
- The analysis goal is fine-mapping variant-level causality rather than gene-level
  prioritization; ProGeM outputs candidate mediator genes, not causal variant probabilities
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: ProGeM
  role: Core R pipeline that identifies bottom-up (proximal/distal), top-down and
    cooccurring candidate causal genes for each sentinel variant
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime environment; pipeline is executed by sourcing ProGeM_settings.R after
    installing Bioconductor/CRAN dependencies
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:f1197dec83f89888ad4873e486559295909158aed9c676c226a37114506bbb99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sentinel-variant-gene-mapping

## Summary

Prioritize candidate causal genes at molecular QTLs using ProGeM, a configurable R pipeline that combines a 'bottom-up' component (nearest genes, LD-region-overlapping genes, and cis-eQTL target genes from GTEx for each sentinel) with a 'top-down' metabolic-gene annotation component, benchmarked to recover 223/227 curated mQTL causal genes (98% sensitivity).

## When to use

Use when you have sentinel variants from molecular QTL mapping (e.g. mQTLs or cis-pQTLs) with GRCh37 coordinates and need to prioritize which local genes are causal, using positional, cis-eQTL (GTEx) and metabolic annotation evidence.

## When NOT to use

- The goal is prioritizing causal genes at trans-pQTLs — the article states the top-down strategy adopted here is not directly applicable and 'would call for a different top-down strategy'
- The phenotype is not metabolic-related and no phenotype-specific .RData gene list has been prepared — the supplied annotated_metabolic_genes.RData covers metabolic-related phenotypes only
- The analysis goal is fine-mapping variant-level causality rather than gene-level prioritization; ProGeM outputs candidate mediator genes, not causal variant probabilities

## Inputs

- Tab-separated sentinel variant file with columns rsID, CHR, START, END (GRCh37 coordinates; use chr:start notation when no rsID exists)
- Tab-separated proxy variant file with columns PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2 (e.g. derived from 1000 Genomes)
- Variant Effect Predictor (VEP) .txt output for both sentinels and proxies, with identifiable sentinel and proxy rsID columns and IMPACT annotation
- GTEx v7 cis-eQTL data (GTEx_Analysis_v7_eQTL.tar.gz) and ProGeM resource files (ProGeM_settings.R, ProGeM_functions.R, ProGeM_commands.R, GRCh37_genes.RData, annotated_metabolic_genes.RData)

## Outputs

- OUTPUT_bottom_up_summary.txt — per-gene bottom-up candidate annotation (nearest/second/third nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead/proxy IMPACT as 0/1/2)
- OUTPUT_top_down_scores.txt — local genes annotated against GO, KEGG, MGI, Orphanet and Reactome with a summed score out of 5
- OUTPUT_cooccurring_candidates.txt — genes that are both bottom-up and top-down candidates, treated as primary causal candidates

## How to apply

Prepare three tab-separated inputs: a sentinel file (rsID, CHR, START, END in GRCh37), a proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2), and a VEP output covering sentinels and proxies (one row per sentinel-proxy combination, with sentinel rsID also entered in the proxy column for sentinel rows). Download GTEx v7 cis-eQTL data (GTEx_Analysis_v7_eQTL.tar.gz), install the GenomicAlignments, GenomicFeatures, GenomicRanges, biomaRt and data.table R packages, then set parameters in ProGeM_settings.R: ±500 kb local window, protein-coding biotype filter, proxy r2 threshold of 0.8, 3 nearest genes, 5 kb LD-region overhang, and all 44 GTEx tissues. Source ProGeM_settings.R to run the full pipeline (~12-15 minutes for ~213 sentinels). Parse OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt and OUTPUT_cooccurring_candidates.txt; treat cooccurring (bottom-up AND top-down) genes as primary causal candidates and score top-down genes by how many of the 5 databases (GO, KEGG, Orphanet, Reactome, MGI) flag them.

## Related tools

- **ProGeM** (Core R pipeline that identifies bottom-up (proximal/distal), top-down and cooccurring candidate causal genes for each sentinel variant) — https://github.com/ds763/ProGeM
- **R** (Runtime environment; pipeline is executed by sourcing ProGeM_settings.R after installing Bioconductor/CRAN dependencies) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R")
```

## Evaluation signals

- Benchmark against the 227 curated mQTL reference set (Supplementary Table S1): sensitivity must equal 223/227 (~98%) and specificity 0.502
- OUTPUT_bottom_up_summary.txt rows must carry correct boolean flags: columns 5-10 are 0/1, and columns 11-12 (lead_IMPACT, proxy_IMPACT) map to 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH
- OUTPUT_top_down_scores.txt score column must equal the row sum of the GO, KEGG, MGI, orphanet and reactome columns (out of 5)
- Proxies in the LD region should be reduced per the r2 threshold: only proxies with r2 >= 0.8 retained when pre-filtering is not applied
- Runtime sanity check: full processing of ~213 sentinel variants should complete in roughly 12-15 minutes on a standard desktop

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of the nearest genes
- ProGeM and the curated mQTL reference set are not entirely independent (e.g. KEGG was used both by ProGeM and as a guide for the expert literature review), so observed mQTL sensitivity may be inflated
- cis-eQTL evidence is a weak positive signal: cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment
- Whether genomic proximity (69% of causal candidates were nearest to the sentinel) generalizes to other molecular QTLs and complex traits requires further research, dependent on availability of high-confidence reference datasets

## Evidence

- [abstract] 98% sensitivity benchmark: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [methods] proxy r2 filter: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [methods] GTEx tissue parameter: "The user can specify the tissues of interest (default is all GTEx tissues, n=44)"
- [readme] nearest gene default: "ProGeM will identify the nearest genes to each sentinel variant.  The user can specify how many nearest genes they would like to pull out (default is 3)"
- [discussion] sensitivity caveat: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [readme] execution mode: "This file can then be saved and sourced, and ProGeMM will then run in its entirety"
