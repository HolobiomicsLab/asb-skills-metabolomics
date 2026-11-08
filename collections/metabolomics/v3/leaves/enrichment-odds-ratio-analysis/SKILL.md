---
name: enrichment-odds-ratio-analysis
description: Use when you have ProGeM bottom-up summary output plus high-confidence
  causal gene assignments (e.g. 227 mQTLs or 562 cis-pQTLs) and need to quantify which
  criteria (nearest gene, VEP IMPACT, cis-eQTL) enrich for true causal genes.
when_to_use_negative:
- You have no high-confidence reference causal gene assignments, since enrichment
  ORs against unrepresentative or biased ground truth (e.g. nearest-gene-biased curation)
  may be inflated
- The QTLs of interest are trans-pQTLs, for which ProGeM's local/top-down prioritization
  strategy is not directly applicable
- You need fine-mapped causal probabilities rather than binary criterion-flag enrichment
  (this skill tests categorical criteria, not posterior causality)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Generates the bottom-up summary flags and VEP IMPACT annotations that feed
    the 2x2 Fisher's exact contingency tables
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runs ProGeM via ProGeM_settings.R and computes Fisher's exact odds ratios
    and confidence intervals
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:50eeb77b445b134d309ba978300a45a5f17bcea07b73106a789827034ecdccf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# enrichment-odds-ratio-analysis

## Summary

Computes Fisher's exact enrichment odds ratios quantifying how well bottom-up annotation criteria (nearest gene, moderate/high VEP IMPACT variant, cis-eQTL target) flag true positive causal genes at molecular QTLs, benchmarked against high-confidence reference assignments such as the 562 cis-pQTLs.

## When to use

Use when you have ProGeM bottom-up summary output plus high-confidence causal gene assignments (e.g. 227 mQTLs or 562 cis-pQTLs) and need to quantify which criteria (nearest gene, VEP IMPACT, cis-eQTL) enrich for true causal genes.

## When NOT to use

- You have no high-confidence reference causal gene assignments, since enrichment ORs against unrepresentative or biased ground truth (e.g. nearest-gene-biased curation) may be inflated
- The QTLs of interest are trans-pQTLs, for which ProGeM's local/top-down prioritization strategy is not directly applicable
- You need fine-mapped causal probabilities rather than binary criterion-flag enrichment (this skill tests categorical criteria, not posterior causality)

## Inputs

- Tab-separated sentinel variant .txt file (rsID, CHR, START, END on GRCh37) for 562 high-confidence cis-pQTL sentinels (Supplementary Table S2)
- Tab-separated proxy variant .txt file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2)
- VEP output .txt on GRCh37 covering all sentinel-proxy combinations, with declared sentinel/proxy rsID column indices
- GTEx_Analysis_v7_eQTL.tar.gz cis-eQTL data (GTEx v7)
- High-confidence causal gene assignments used as ground truth (227 mQTLs from Supplementary Table S1 and/or 562 cis-pQTLs)

## Outputs

- OUTPUT_bottom_up_summary.txt with per-gene boolean flags (nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL) and VEP IMPACT scores
- Fisher's exact enrichment odds ratios with 95% confidence intervals for the nearest-gene criterion (expected OR = 54 [41-71]) and the moderate-impact criterion (expected OR = 99 [37-374])
- Supporting lead/proxy cis-eQTL and VEP IMPACT annotation files

## How to apply

Prepare a tab-separated sentinel file of the 562 high-confidence cis-pQTL rsIDs with GRCh37 start/end coordinates, run VEP on sentinels and proxies on GRCh37 (one record per sentinel-proxy combination), and configure ProGeM_settings.R with all bottom-up criteria enabled (nearest/second/third-nearest gene, LD-overlapping genes, cis-eQTL targets against GTEx v7) and proxy r2 threshold 0.8. Source ProGeM_settings.R in R to generate OUTPUT_bottom_up_summary.txt, whose boolean columns (nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL) and IMPACT columns (0=MODIFIER/LOW, 1=MODERATE, 2=HIGH) encode each criterion per candidate gene. Then build 2x2 contingency tables flagging whether the true causal gene is (a) the nearest gene and (b) harbours a moderate/high-impact sentinel or proxy variant, and compute Fisher's exact odds ratios with 95% confidence intervals against the 562 reference assignments. The rationale is that genomic proximity was the most reliable indicator of true positives, so its enrichment should dominate over cis-eQTL target evidence, which produced roughly three false positives per true positive.

## Related tools

- **ProGeM** (Generates the bottom-up summary flags and VEP IMPACT annotations that feed the 2x2 Fisher's exact contingency tables) — https://github.com/ds763/ProGeM
- **R** (Runs ProGeM via ProGeM_settings.R and computes Fisher's exact odds ratios and confidence intervals) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R")  # then compute Fisher exact ORs on OUTPUT_bottom_up_summary.txt flags: fisher.test(table(true_gene==nearest_gene, true_gene %in% genes))
```

## Evaluation signals

- Recomputed nearest-gene enrichment OR equals 54 [41-71], P = 4.1 x 10^-251 on the 562 cis-pQTLs
- Recomputed moderate-impact enrichment OR equals 99 [37-374] on the 562 cis-pQTLs
- mQTL cross-validation nearest-gene OR equals 30 [20-45], P = 2.4 x 10^-82, confirming proximity enrichment replicates across datasets
- OUTPUT_bottom_up_summary.txt contains exactly the 12 documented columns with boolean values 0/1 in columns 5-10 and IMPACT values {0,1,2} in columns 11-12
- 69% of causal candidates should be nearest to the sentinel variant when recomputed on the 227 mQTL reference set

## Limitations

- Reference causal gene assignments at the 227 mQTLs could have been subject to bias, such as prioritization of the nearest genes, inflating the proximity enrichment
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent, so observed sensitivity at mQTLs may be inflated
- Whether genomic proximity is a good indicator of true positive causal genes for other molecular QTLs and complex disease traits requires further research and depends on availability of high-confidence reference datasets

## Evidence

- [results] In cross-validation, the nearest-gene criterion achieved the highest significance for both datasets, including the 562 cis-pQTLs (OR = 54 [41-71], P = 4.1 x 10^-251: [evidence span withheld — anchor status: not_found]
- [abstract] 69% nearest-gene enrichment: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of"
- [abstract] cis-eQTL false positives: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [readme] bottom-up summary columns and IMPACT coding: "columns 11-12 are nominal variables referring to VEP IMPACT annotation with 0=MODIFIER or LOW IMPACT, 1=MODERATE IMPACT, 2=HIGH IMPACT"
- [methods] proxy r2 filter: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
