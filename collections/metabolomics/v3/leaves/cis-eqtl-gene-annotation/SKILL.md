---
name: cis-eqtl-gene-annotation
description: Use when sentinel and proxy variant lists (rsID + GRCh37 coordinates)
  at molecular QTLs require cis-eQTL target gene annotation from GTEx v7 across user-specified
  tissues as part of a bottom-up candidate-gene prioritization.
when_to_use_negative:
- Input loci are trans-pQTLs or distal signals — the article notes the top-down strategy
  adopted (metabolic-gene databases) is not directly applicable to trans-pQTLs and
  a different approach would be needed
- The goal is a definitive causal-gene assignment from eQTL evidence alone — cis-eQTL
  target genes were found to be relatively poor indicators of true positive causal
  genes (about 3 false positives per true positive)
- Variants are not on GRCh37 — the sentinel and proxy input files require GRCh37 start
  and end coordinates
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ProGeM
  role: Pipeline that searches GTEx v7 cis-eQTL data to flag eQTL target genes of
    sentinels and proxies in the bottom-up summary
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime environment; ProGeM is a configurable R script sourced via ProGeM_settings.R
    (requires GenomicRanges, GenomicFeatures, GenomicAlignments, biomaRt, data.table)
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:eb1d5c6ae932aea0fa1fe7737eaf3ab113a5e13891417e5cddf7dcc53e08aad2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - biomaRt
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cis-eqtl-gene-annotation

## Summary

Annotates candidate causal genes at molecular QTLs by identifying cis-eQTL target genes of sentinel and proxy variants from GTEx v7 data, as one of the bottom-up criteria in the ProGeM framework, producing boolean lead_eQTL/proxy_eQTL flags in the bottom-up summary output.

## When to use

Use when sentinel and proxy variant lists (rsID + GRCh37 coordinates) at molecular QTLs require cis-eQTL target gene annotation from GTEx v7 across user-specified tissues as part of a bottom-up candidate-gene prioritization.

## When NOT to use

- Input loci are trans-pQTLs or distal signals — the article notes the top-down strategy adopted (metabolic-gene databases) is not directly applicable to trans-pQTLs and a different approach would be needed
- The goal is a definitive causal-gene assignment from eQTL evidence alone — cis-eQTL target genes were found to be relatively poor indicators of true positive causal genes (about 3 false positives per true positive)
- Variants are not on GRCh37 — the sentinel and proxy input files require GRCh37 start and end coordinates

## Inputs

- Tab-separated sentinel variant file (rsID, CHR, START, END; GRCh37 coordinates)
- Tab-separated proxy variant file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2)
- GTEx cis-eQTL data folder (GTEx_Analysis_v7_eQTL.tar.gz) with significant SNP-gene associations per tissue
- ProGeM_settings.R configuration file (tissue selection, r2 threshold, GTEx data path)

## Outputs

- OUTPUT_bottom_up_summary.txt with per-gene boolean lead_eQTL and proxy_eQTL flags (0/1) alongside nearest/second_nearest/third_nearest, LD_overlapping, and VEP IMPACT columns

## How to apply

Download the GTEx cis-eQTL dataset (GTEx_Analysis_v7_eQTL.tar.gz from gtexportal.org, 728 MB, containing eGene and significant permutation-based SNP-gene associations) and prepare the tab-separated sentinel file (rsID, CHR, START, END in GRCh37) and proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2); proxies are pre-filtered by a user-defined r2 threshold (default 0.8). In ProGeM_settings.R, declare the GTEx data directory, the column indices of the VEP output, and the tissues of interest (default is all GTEx tissues, n=44), then source the settings file so the pipeline searches GTEx v7 data to identify cis-eQTL target genes of sentinel and proxy variants. Output is written to OUTPUT_bottom_up_summary.txt as boolean lead_eQTL and proxy_eQTL columns (0=FALSE, 1=TRUE) per candidate gene per sentinel. Interpret eQTL-target evidence cautiously: in the article's benchmarking, cis-eQTL data led to roughly three false positive causal-gene assignments for every one true positive, so treat eQTL flags as supporting rather than definitive evidence relative to proximity (nearest-gene) criteria.

## Related tools

- **ProGeM** (Pipeline that searches GTEx v7 cis-eQTL data to flag eQTL target genes of sentinels and proxies in the bottom-up summary) — https://github.com/ds763/ProGeM
- **R** (Runtime environment; ProGeM is a configurable R script sourced via ProGeM_settings.R (requires GenomicRanges, GenomicFeatures, GenomicAlignments, biomaRt, data.table)) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R")  # after setting GTEx v7 path, tissue list (default all 44), and proxy r2 threshold 0.8
```

## Evaluation signals

- lead_eQTL and proxy_eQTL columns in OUTPUT_bottom_up_summary.txt are strictly boolean (0=FALSE, 1=TRUE) for every candidate gene row
- All proxies surviving annotation satisfy the user-defined r2 threshold (default 0.8)
- eQTL-flagged candidate sets at the 227 curated mQTLs should reproduce the reported enrichment pattern of ~3 false positives per true positive for the cis-eQTL criterion, while proximity (nearest gene) reaches OR = 30 [20-45] at mQTLs and OR = 54 [41-71] at cis-pQTLs
- At exemplar loci, expected per-locus calls must hold: MTHFR and AOC1 as sole cooccurring candidates for rs1801133 and rs1005390, and G6PD absent from rs766420 candidates
- Full ProGeM run over ~213 sentinels completes in ~12-15 minutes on a standard desktop (4 GB RAM), confirming GTEx eQTL search completed without data-path errors

## Limitations

- cis-eQTL target genes (GTEx v7) are relatively poor indicators of true positive causal genes: about three false positive assignments for every true positive
- eQTL annotation is only as informative as the tissue list; the default of all GTEx tissues (n=44) maximizes sensitivity but increases false positives
- Reference assignments at the 227 mQTLs may be biased toward nearest genes, and ProGeM is not independent of the curated mQTL dataset, so observed sensitivity may be inflated

## Evidence

- [readme] GTEx cis-eQTL search: "ProGeM will search GTEx v6p data to identify cis-eQTL target genes of sentinel and proxy variants."
- [methods] GTEx data download: "Download a zipped folder containing cis-eQTL data (v7) from the Genotype-Tissue Expression (GTEx) project"
- [abstract] cis-eQTL criterion performance: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [readme] tissue default: "The user can specify the tissues of interest (default is all GTEx tissues, n=44)."
- [discussion] poor indicator caveat: "cis-eQTL target genes (as reported in GTEx v7 data) as relatively poor indicators of ‘true positive’ causal genes"
