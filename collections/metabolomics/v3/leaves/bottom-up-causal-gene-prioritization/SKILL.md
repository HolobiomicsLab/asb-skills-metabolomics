---
name: bottom-up-causal-gene-prioritization
description: Use when you have sentinel variants (with GRCh37 coordinates and optional
  LD proxies) at molecular QTLs and need to nominate candidate causal genes positionally
  and functionally, then benchmark those nominations against a high-confidence causal
  gene reference set.
when_to_use_negative:
- 'The QTL is trans (e.g. trans-pQTL): ProGeM''s top-down proximity-based strategy
  does not directly apply and a different approach would be needed'
- No high-confidence reference causal gene assignments are available, so enrichment-based
  validation of the bottom-up criteria cannot be performed
- You need literature/text-mining or database-knowledge prioritisation of genes with
  no positional link to a sentinel variant (use top-down/complementary tools such
  as Phenolyzer instead)
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Highly configurable R pipeline that annotates bottom-up candidate causal genes
    (nearest, LD-overlapping, cis-eQTL, VEP IMPACT) at molecular QTLs and writes the
    bottom-up summary table
  repo: https://github.com/ds763/ProGeM
- name: R
  role: 'Runtime environment: ProGeM settings file is sourced in R; GenomicRanges/GenomicFeatures/GenomicAlignments/biomaRt/data.table
    packages are required and Fisher''s exact tests are computed in R'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:55ffd820573a1aa4f9ee1b1e3f7e7dede6418ef127b2440fd11990b95c55ab16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bottom-up-causal-gene-prioritization

## Summary

Prioritises candidate causal genes at molecular QTLs using ProGeM's bottom-up criteria — nearest genes, LD-overlapping genes, cis-eQTL target genes, and VEP HIGH/MODERATE IMPACT variants — and validates them by Fisher's exact enrichment against high-confidence causal gene assignments.

## When to use

Use when you have sentinel variants (with GRCh37 coordinates and optional LD proxies) at molecular QTLs and need to nominate candidate causal genes positionally and functionally, then benchmark those nominations against a high-confidence causal gene reference set.

## When NOT to use

- The QTL is trans (e.g. trans-pQTL): ProGeM's top-down proximity-based strategy does not directly apply and a different approach would be needed
- No high-confidence reference causal gene assignments are available, so enrichment-based validation of the bottom-up criteria cannot be performed
- You need literature/text-mining or database-knowledge prioritisation of genes with no positional link to a sentinel variant (use top-down/complementary tools such as Phenolyzer instead)

## Inputs

- Tab-separated sentinel variant .txt file: rsID, CHR, START, END (GRCh37 coordinates; use chr:start notation if no rsID)
- Tab-separated proxy variant .txt file: PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2
- VEP output .txt on GRCh37 for sentinels and proxies, one row per sentinel-proxy combination, with sentinel and proxy rsID columns
- GTEx v7 cis-eQTL data (GTEx_Analysis_v7_eQTL.tar.gz) with significant SNP-gene associations
- GRCh37_genes.RData gene model (from Homo_sapiens.GRCh37.82.gtf)
- High-confidence causal gene assignments (e.g. 562 cis-pQTL sentinels from Supplementary Table S2)

## Outputs

- OUTPUT_bottom_up_summary.txt: per-candidate-gene table with boolean flags (nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL) and lead/proxy IMPACT scores (0=LOW/MODIFIER, 1=MODERATE, 2=HIGH)
- Supporting lead/proxy cis-eQTL annotation files and VEP IMPACT annotation files
- Fisher's exact enrichment odds ratios with confidence intervals (e.g. nearest-gene OR=54 [41-71], moderate-impact OR=99 [37-374] at 562 cis-pQTLs)

## How to apply

Prepare a tab-separated sentinel file (rsID, CHR, START, END in GRCh37) and a six-column proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2), plus VEP output on GRCh37 with one record per sentinel-proxy combination and sentinel/proxy rsID column indices declared in ProGeM_settings.R. In ProGeM_settings.R enable all bottom-up criteria (first three nearest genes, LD-overlapping genes with 5 kb overhang, cis-eQTL target genes searched against GTEx v7 eQTL across specified tissues, default all 44), set the proxy r2 threshold to 0.8, then source the settings file to run ProGeM and generate OUTPUT_bottom_up_summary.txt. From the summary's boolean flags (nearest/second_nearest/third_nearest/LD_overlapping/lead_eQTL/proxy_eQTL) and VEP IMPACT coding (0=LOW/MODIFIER, 1=MODERATE, 2=HIGH), compute Fisher's exact enrichment odds ratios for the true causal gene being nearest and for harbouring a moderate-impact variant. Rationale: genomic proximity is the most reliable indicator of true positive causal genes, whereas cis-eQTL data yield roughly three false positives per true positive, so enrichment statistics — not the cis-eQTL flag alone — establish credibility.

## Related tools

- **ProGeM** (Highly configurable R pipeline that annotates bottom-up candidate causal genes (nearest, LD-overlapping, cis-eQTL, VEP IMPACT) at molecular QTLs and writes the bottom-up summary table) — https://github.com/ds763/ProGeM
- **R** (Runtime environment: ProGeM settings file is sourced in R; GenomicRanges/GenomicFeatures/GenomicAlignments/biomaRt/data.table packages are required and Fisher's exact tests are computed in R)

## Examples

```
source("ProGeM_settings.R")  # after configuring sentinel/proxy files, VEP output paths, r2=0.8 and GTEx v7 tissue list
```

## Evaluation signals

- Recomputed nearest-gene enrichment against the 562 high-confidence cis-pQTL assignments equals OR=54 [41-71], P=4.1e-251
- Recomputed moderate-impact enrichment equals OR=99 [37-374] within its reported confidence interval
- OUTPUT_bottom_up_summary.txt has the expected 12-column schema (LEAD_rsID ... proxy_IMPACT) with boolean 0/1 flags in columns 5-10 and IMPACT coding 0/1/2 in columns 11-12
- Proxies are correctly filtered at the user-defined r2 threshold (default 0.8) before annotation
- VEP output contains exactly one record per sentinel-proxy combination, with sentinel rsID entered in both rsID columns for sentinel rows

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be biased toward nearest genes, and ProGeM/the curated mQTL dataset are not fully independent, so sensitivity at mQTLs may be inflated
- cis-eQTL target genes are relatively poor indicators of true positive causal genes (~3 false positives per true positive)
- Whether genomic proximity generalises to other molecular QTLs and complex traits depends on availability of high-confidence reference datasets
- Runtime ~12-15 minutes for 213 sentinels on a 4GB RAM dual-core desktop; larger sentinel sets scale accordingly

## Evidence

- [results] In cross-validation, the nearest-gene criterion achieved the highest significance for both datasets, including the 562 cis-pQTLs (OR = 54 [41-71], P = 4.1 x 10^-251: [evidence span withheld — anchor status: not_found]
- [methods] tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of your sentinel variants: "A tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of your sentinel variants"
- [readme] ProGeM will filter based on a user-defined r2 threshold (default r2 is 0.8): "If proxies have not been pre-filtered prior to input, then ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8) if desired."
- [abstract] 69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [abstract] cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
