---
name: vep-impact-consequence-annotation
description: Use when prioritizing candidate causal genes at molecular QTLs and you
  need variant-level functional consequence evidence (VEP IMPACT) to mark genes hit
  by sentinels or LD proxies.
when_to_use_negative:
- The variant set is trans-QTL or lacks proxy/LD information — the sentinel-proxy
  IMPACT annotation scheme assumes LD-defined proxies for each sentinel
- Variants are not on GRCh37 coordinates — VEP must be run against the GRCh37 assembly
  for ProGeM's gene model (Homo_sapiens.GRCh37.82.gtf) to align
- The goal is top-down candidate scoring only — IMPACT annotation feeds the bottom-up
  component and is redundant if no positional/cis-eQTL candidates are being evaluated
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Consumes the VEP output to annotate bottom-up candidate mediating genes with
    HIGH/MODERATE IMPACT flags and writes the bottom-up summary tables
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Host environment in which the ProGeM R script parses VEP output columns (data.table)
    and integrates IMPACT annotations with GenomicRanges-based gene candidates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:196703083b525abd797f8c874ff60ee7a77bd52c81aca950fd0b18eedd67e4fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - GenomicAlignments
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - biomaRt
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# vep-impact-consequence-annotation

## Summary

Annotate sentinel and proxy variants of molecular QTLs with Ensembl Variant Effect Predictor (VEP) IMPACT categories (HIGH/MODERATE/LOW/MODIFIER) and use these to flag bottom-up candidate causal genes whose function the variant is expected to alter.

## When to use

Use when prioritizing candidate causal genes at molecular QTLs and you need variant-level functional consequence evidence (VEP IMPACT) to mark genes hit by sentinels or LD proxies.

## When NOT to use

- The variant set is trans-QTL or lacks proxy/LD information — the sentinel-proxy IMPACT annotation scheme assumes LD-defined proxies for each sentinel
- Variants are not on GRCh37 coordinates — VEP must be run against the GRCh37 assembly for ProGeM's gene model (Homo_sapiens.GRCh37.82.gtf) to align
- The goal is top-down candidate scoring only — IMPACT annotation feeds the bottom-up component and is redundant if no positional/cis-eQTL candidates are being evaluated

## Inputs

- Tab-separated .txt of sentinel variants (rsID, CHR, START, END on GRCh37)
- Tab-separated .txt of proxy variants (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2) filtered at r2 >= 0.8 by default
- VEP output .txt covering all sentinel-proxy combinations, with configurable sentinel- and proxy-rsID column indices

## Outputs

- OUTPUT_bottom_up_summary.txt with lead_IMPACT and proxy_IMPACT columns coded 0 (MODIFIER/LOW), 1 (MODERATE), 2 (HIGH)
- OUTPUT_VEP_IMPACT_annotations.txt standalone VEP IMPACT annotation table

## How to apply

Run VEP (GRCh37 build, e.g. via grch37.ensembl.org) on both sentinel and proxy variants, producing a tab-separated .txt output with one record per sentinel-proxy combination — if a proxy serves multiple sentinels, emit a row per pair. Configure ProGeM_settings.R with the column indices holding sentinel and proxy rsIDs; for sentinel rows, the sentinel rsID is entered in both columns. Parse the IMPACT column, mapping variant CONSEQUENCE terms to the four impact tiers (e.g. stop_gained/frameshift_variant are HIGH; missense_variant/inframe_deletion are MODERATE; synonymous_variant is LOW; intron_variant/upstream_gene_variant are MODIFIER) and recode 0=MODIFIER or LOW, 1=MODERATE, 2=HIGH. Flag any bottom-up candidate gene containing a sentinel or proxy variant with HIGH or MODERATE IMPACT, writing lead_IMPACT and proxy_IMPACT columns to OUTPUT_bottom_up_summary.txt and a standalone OUTPUT_VEP_IMPACT_annotations.txt. Rationale: functional disruption of the gene by the variant is one of the two assumptions (bottom-up) a causal mediating gene must satisfy, complementing the top-down metabolic-annotation component.

## Related tools

- **ProGeM** (Consumes the VEP output to annotate bottom-up candidate mediating genes with HIGH/MODERATE IMPACT flags and writes the bottom-up summary tables) — https://github.com/ds763/ProGeM
- **R** (Host environment in which the ProGeM R script parses VEP output columns (data.table) and integrates IMPACT annotations with GenomicRanges-based gene candidates)

## Examples

```
source("ProGeM_settings.R"); source("ProGeM_functions.R"); source("ProGeM_commands.R")  # settings point VEP columns at sentinel/proxy rsID indices
```

## Evaluation signals

- Every VEP record/row corresponding directly to a sentinel variant has the sentinel rsID in both the sentinel and proxy rsID columns
- Each proxy variant appearing under multiple sentinels yields one VEP record per sentinel-proxy combination
- lead_IMPACT/proxy_IMPACT values are restricted to the nominal codes 0, 1, 2 (0=MODIFIER or LOW, 1=MODERATE, 2=HIGH)
- Every bottom-up candidate flagged HIGH/MODERATE has a matching VEP row whose IMPACT column is literally HIGH or MODERATE
- OUTPUT_VEP_IMPACT_annotations.txt row count is consistent with the number of sentinel-proxy pairs passing the r2 >= 0.8 filter

## Limitations

- VEP parameters are user-chosen, so IMPACT annotations are not guaranteed comparable across runs unless the same assembly, cache and consequence set are used
- ProGeM only highlights HIGH or MODERATE IMPACT variants — LOW and MODIFIER consequences carry no candidate-flagging weight
- cis-eQTL and proximity evidence dominate ProGeM's candidate lists; VEP IMPACT is a supplementary annotation and cis-eQTL data alone produced ~3 false positives per true positive in the article's benchmark

## Evidence

- [readme] ProGeM will also annotate bottom-up candidates with information from a user-supplied Variant Effect Predictor (VEP) output: "ProGeM will also annotate bottom-up candidates with information from a user-supplied Variant Effect Predictor (VEP) output"
- [methods] ProGeM highlights HIGH/MODERATE bottom-up candidates: "ProGeM will highlight any bottom-up candidate mediating genes that contain either a sentinel or a proxy variant expected to exert a HIGH or a MODERATE IMPACT"
- [readme] VEP output format requirements: "in the VEP output there should be a separate record/row for each sentinel-proxy combination"
- [readme] IMPACT column coding in the bottom-up summary: "columns 11-12 are nominal variables referring to VEP IMPACT annotation with 0=MODIFIER or LOW IMPACT, 1=MODERATE IMPACT, 2=HIGH IMPACT"
- [methods] ProGeM annotates bottom-up candidates with VEP IMPACT: [evidence span withheld — anchor status: not_found]
