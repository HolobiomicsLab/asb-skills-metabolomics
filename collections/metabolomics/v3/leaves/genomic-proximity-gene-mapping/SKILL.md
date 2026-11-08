---
name: genomic-proximity-gene-mapping
description: Use when you have a set of sentinel variants (e.g. mQTLs or cis-pQTLs
  with GRCh37 coordinates) and need to prioritize or validate causal gene assignments
  based on genomic proximity, LD-overlap, or cis-eQTL targeting.
when_to_use_negative:
- The goal is prioritization at trans-QTLs — the article notes the top-down strategy
  is not applicable to trans-pQTLs and would need a different approach
- No high-confidence reference causal gene assignments exist for the QTL set, since
  the proximity/enrichment validation depends on their availability
- You need functional annotation prioritization only (top-down metabolic scoring),
  not positional mapping of sentinels to nearest/LD-overlapping genes
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Configurable R pipeline that annotates bottom-up candidate causal genes by
    nearest-gene, LD-overlap, cis-eQTL and VEP IMPACT criteria and writes OUTPUT_bottom_up_summary.txt
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime environment for sourcing ProGeM_settings.R and computing Fisher's
    exact enrichment odds ratios
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:bb8a74cd8bd6acf63d059e9d017595adb9f7722322bff0eebff4cc7da65a686b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# genomic-proximity-gene-mapping

## Summary

Maps sentinel molecular QTL variants to their nearest candidate causal genes using ProGeM, then quantifies enrichment of the nearest-gene criterion against high-confidence causal gene assignments via Fisher's exact tests. Genomic proximity is treated as the most reliable indicator of true positive causal genes.

## When to use

Use when you have a set of sentinel variants (e.g. mQTLs or cis-pQTLs with GRCh37 coordinates) and need to prioritize or validate causal gene assignments based on genomic proximity, LD-overlap, or cis-eQTL targeting.

## When NOT to use

- The goal is prioritization at trans-QTLs — the article notes the top-down strategy is not applicable to trans-pQTLs and would need a different approach
- No high-confidence reference causal gene assignments exist for the QTL set, since the proximity/enrichment validation depends on their availability
- You need functional annotation prioritization only (top-down metabolic scoring), not positional mapping of sentinels to nearest/LD-overlapping genes

## Inputs

- Tab-separated sentinel variant .txt file (rsID | CHR | START | END on GRCh37, e.g. rs1801133 | 1 | 11856378 | 11856378)
- Tab-separated proxy variant .txt file (PROXY_rsID | PROXY_CHR | PROXY_START | PROXY_END | LEAD_rsID | r2), optionally pre-filtered or filtered by ProGeM at r2 = 0.8
- VEP output .txt on GRCh37 covering sentinels and proxies, one row per sentinel-proxy combination, with declared sentinel/proxy rsID column indices
- GTEx v7 cis-eQTL data (GTEx_Analysis_v7_eQTL.tar.gz)
- High-confidence causal gene assignments (e.g. Supplementary Table S2 for 562 cis-pQTLs) for validation

## Outputs

- OUTPUT_bottom_up_summary.txt with boolean nearest/second_nearest/third_nearest/LD_overlapping/lead_eQTL/proxy_eQTL flags and lead_IMPACT/proxy_IMPACT annotations (0=MODIFIER or LOW, 1=MODERATE, 2=HIGH)
- Supporting lead/proxy cis-eQTL target gene files
- Fisher's exact enrichment odds ratios with 95% CIs (e.g. cis-pQTL nearest-gene OR = 54 [41-71]; moderate-impact OR = 99 [37-374])

## How to apply

Prepare a tab-separated sentinel input file with rsIDs, chromosomes, and GRCh37 start/end coordinates, plus a proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2) filtered at an r2 threshold of 0.8. Run Ensembl VEP on GRCh37 with one record per sentinel-proxy combination and declare the sentinel/proxy rsID column indices in ProGeM_settings.R. Enable all bottom-up criteria (nearest, second-nearest, third-nearest gene, LD-overlapping genes with default 5 kb overhang, and cis-eQTL targets searched against GTEx v7 eQTL data across user-specified tissues, default all 44) and source ProGeM_settings.R to generate OUTPUT_bottom_up_summary.txt. Using the boolean flags (nearest/second_nearest/third_nearest, lead_eQTL/proxy_eQTL, lead_IMPACT/proxy_IMPACT), compute Fisher's exact odds ratios for (a) the true causal gene being nearest and (b) the true gene harbouring a MODERATE- or HIGH-impact sentinel/proxy variant against the high-confidence assignments — the rationale being that proximity is the strongest positional indicator (69% of causal candidates were nearest; cis-eQTL data gave ~3 false positives per true positive).

## Related tools

- **ProGeM** (Configurable R pipeline that annotates bottom-up candidate causal genes by nearest-gene, LD-overlap, cis-eQTL and VEP IMPACT criteria and writes OUTPUT_bottom_up_summary.txt) — https://github.com/ds763/ProGeM
- **R** (Runtime environment for sourcing ProGeM_settings.R and computing Fisher's exact enrichment odds ratios)

## Examples

```
source("ProGeM_settings.R")  # after configuring sentinels.txt, proxies.txt, VEP output and GTEx v7 paths; outputs OUTPUT_bottom_up_summary.txt
```

## Evaluation signals

- Recomputed nearest-gene enrichment OR on the 562 cis-pQTLs equals 54 [41-71], P = 4.1 x 10^-251
- Recomputed moderate-impact variant enrichment OR equals 99 [37-374]
- OUTPUT_bottom_up_summary.txt has 12 columns with boolean flags in columns 5-10 and IMPACT coding 0/1/2 in columns 11-12
- Proxy list retains only variants meeting the r2 >= 0.8 threshold when pre-filtering is not applied
- ProGeM misses at most 4 of the 227 mQTL reference causal genes (98% sensitivity benchmark)

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be biased toward nearest genes, and ProGeM/its curated dataset are not fully independent, potentially inflating observed mQTL sensitivity
- cis-eQTL targeting is a weak indicator, yielding roughly three false positives per true positive
- Whether genomic proximity generalizes to other molecular QTLs and complex traits requires further high-confidence reference datasets

## Evidence

- [abstract] Proximity as strongest indicator: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator"
- [readme] Sentinel input format: "A tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of your sentinel variants of interest across four columns"
- [readme] Proxy r2 filtering: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8) if desired"
- [abstract] cis-eQTL false positives: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [discussion] Reference dataset bias caveat: "the preparation of the reference causal gene assignments at the 227 mQTL, which could have been subject to bias, such as the prioritization of the nearest genes"
