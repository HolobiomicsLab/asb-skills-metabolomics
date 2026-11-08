---
name: vep-variant-impact-annotation
description: Use when sentinel/proxy variants for molecular QTLs need functional consequence
  annotation (VEP IMPACT HIGH/MODERATE/LOW/MODIFIER) as input to bottom-up causal
  gene prioritization.
when_to_use_negative:
- The analysis targets trans-pQTLs, where ProGeM's local-window and top-down strategy
  would need a fundamentally different approach
- Sentinel/proxy coordinates are not on GRCh37 — VEP must be run against GRCh37 for
  compatibility with the GRCh37_genes.RData gene model
- 'The goal is regulatory/intergenic interpretation only: VEP MODIFIER consequences
  (e.g. intron_variant, upstream_gene_variant) are scored 0 and not used to highlight
  candidate genes'
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: ProGeM
  role: Consumes the VEP output (column indices of sentinel/proxy rsIDs declared in
    ProGeM_settings.R) and annotates bottom-up candidates by VEP IMPACT HIGH/MODERATE
    status
  repo: https://github.com/ds763/ProGeM
- name: VEP
  role: Ensembl Variant Effect Predictor run on GRCh37 over sentinels and proxies
    to produce the IMPACT-classified annotation file
- name: R
  role: Runs the ProGeM pipeline (GenomicRanges, GenomicFeatures, GenomicAlignments,
    biomaRt, data.table) that parses the VEP output
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:946454b8cc4eb3c8284d51e071b83c81dd0e7761553ff259fb0d919c62c15898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - VEP
  - biomaRt
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# vep-variant-impact-annotation

## Summary

Run Ensembl Variant Effect Predictor (VEP) on molecular QTL sentinel and proxy variants and feed the IMPACT-annotated output into ProGeM so that bottom-up candidate causal genes containing a HIGH or MODERATE IMPACT variant can be highlighted.

## When to use

Use when sentinel/proxy variants for molecular QTLs need functional consequence annotation (VEP IMPACT HIGH/MODERATE/LOW/MODIFIER) as input to bottom-up causal gene prioritization.

## When NOT to use

- The analysis targets trans-pQTLs, where ProGeM's local-window and top-down strategy would need a fundamentally different approach
- Sentinel/proxy coordinates are not on GRCh37 — VEP must be run against GRCh37 for compatibility with the GRCh37_genes.RData gene model
- The goal is regulatory/intergenic interpretation only: VEP MODIFIER consequences (e.g. intron_variant, upstream_gene_variant) are scored 0 and not used to highlight candidate genes

## Inputs

- Tab-separated .txt sentinel file with rsID, CHR, START, END (GRCh37 coordinates)
- Tab-separated .txt proxy file with PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2 columns
- GTEx_Analysis_v7_eQTL.tar.gz (unzipped) for cis-eQTL target gene annotation in the same pipeline

## Outputs

- VEP output .txt file covering sentinels and proxies with one record per sentinel-proxy combination
- OUTPUT_bottom_up_summary.txt with lead_IMPACT and proxy_IMPACT columns (0=MODIFIER/LOW, 1=MODERATE, 2=HIGH)
- Fisher's exact enrichment odds ratios for the moderate/high-impact criterion (e.g. cis-pQTL OR = 99 [37-374])

## How to apply

Prepare tab-separated input files of sentinel variants (rsID, CHR, START, END on GRCh37) and proxies (with LEAD_rsID and r2 columns, filtered at the default r2 threshold 0.8), then run VEP (GRCh37) using both sentinels and proxies as input, ensuring one VEP record per sentinel-proxy combination: a proxy serving multiple sentinels gets multiple rows, and for sentinel rows the sentinel rsID is entered in both the sentinel and proxy rsID columns. In ProGeM_settings.R, declare the column indices of the VEP output that contain the sentinel and proxy rsIDs. ProGeM parses the VEP IMPACT column and flags bottom-up candidate genes whose sentinel or proxy variant is HIGH IMPACT (e.g. stop_gained, frameshift_variant) or MODERATE IMPACT (e.g. missense_variant, inframe_insertion/deletion); the bottom-up summary encodes 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH. This IMPACT annotation feeds enrichment analyses such as the cis-pQTL cross-validation, where the moderate-impact criterion gave OR = 99 [37-374] against 562 high-confidence causal gene assignments.

## Related tools

- **ProGeM** (Consumes the VEP output (column indices of sentinel/proxy rsIDs declared in ProGeM_settings.R) and annotates bottom-up candidates by VEP IMPACT HIGH/MODERATE status) — https://github.com/ds763/ProGeM
- **VEP** (Ensembl Variant Effect Predictor run on GRCh37 over sentinels and proxies to produce the IMPACT-classified annotation file)
- **R** (Runs the ProGeM pipeline (GenomicRanges, GenomicFeatures, GenomicAlignments, biomaRt, data.table) that parses the VEP output) — https://github.com/ds763/ProGeM

## Examples

```
R> source("ProGeM_settings.R")  # with vep.output.column.lead.rsID and vep.output.column.proxy.rsID set to the VEP txt columns
```

## Evaluation signals

- VEP output contains exactly one record/row for each sentinel-proxy combination, with sentinel rsID in both sentinel and proxy columns for direct sentinel rows
- All coordinates are GRCh37 and the sentinel/proxy files use the required four- and six-column tab-separated formats
- OUTPUT_bottom_up_summary.txt IMPACT columns contain only values in {0, 1, 2} consistent with VEP HIGH/MODERATE/LOW/MODIFIER consequences
- Recomputed moderate-impact enrichment against 562 cis-pQTL high-confidence assignments reproduces OR = 99 [37-374]
- ProGeM fully processes e.g. 213 sentinels in ~12-15 minutes on a standard desktop, so gross runtime deviations indicate misconfigured inputs

## Limitations

- IMPACT annotation depends on VEP consequences: MODIFIER/LOW variants (0) do not elevate a candidate gene, so coding-function evidence is limited to the rarer missense/LOF class
- If proxies are not pre-filtered, ProGeM must apply the user-defined r2 threshold (default 0.8), and missing r2 annotation can distort which variants carry HIGH/MODERATE IMPACT evidence
- Moderate/high-impact sentinels are rare (cis-pQTL CI [37-374] is wide), so the impact-based criterion is far less stable than the nearest-gene criterion (OR = 54 [41-71])
- Reference causal gene assignments may themselves be biased towards nearest genes, so impact-based enrichment must be interpreted with that caveat

## Evidence

- [readme] ProGeM will highlight any bottom-up candidate mediating genes that contain either a sentinel or a proxy variant expected to exert a HIGH or a MODERATE IMPACT: "ProGeM will highlight any bottom-up candidate mediating genes that contain either a sentinel or a proxy variant expected to exert a HIGH or a MODERATE IMPACT"
- [readme] VEP one record per sentinel-proxy combination: "there should be a separate record/row for each sentinel-proxy combination, meaning that if a variant is a proxy for more than one sentinel"
- [methods] ProGeM_settings.R column indices for VEP rsID columns: "the output must contain a column with (i) sentinel rsIDs as well as a column with (ii) proxy rsIDs - in the "ProGeM_settings.R" files the user is asked to indicate the column indices"
- [methods] proxy r2 filtering default 0.8: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [discussion] cis-pQTL cross-validation enrichment of impact criterion: "cis-eQTL target genes (as reported in GTEx v7 data) as relatively poor indicators of ‘true positive’ causal genes"
