---
name: sentinel-proxy-ld-variant-filtering
description: Use when you have sentinel variants plus LD proxies and GRCh37 gene coordinates
  and need positional bottom-up candidate causal genes for molecular QTL prioritization.
when_to_use_negative:
- 'The QTL is a trans-pQTL: the paper notes ''the prioritization of candidate causal
  genes at trans-pQTLs would call for a different top-down strategy to the one adopted
  here'', so local positional filtering around a sentinel is not appropriate'
- Input variants lack GRCh37 coordinates or LD proxy information — the nearest-gene
  and LD-overlap logic depends on GRCh37 sentinel/proxy start/end positions and an
  r2 column
- You need only top-down trait-annotation candidates without any positional/variant-level
  evidence — this skill is specifically the bottom-up component
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: R script implementing the sentinel/proxy filtering, nearest-gene extraction
    and LD-range overlap to produce the bottom-up summary
  repo: https://github.com/ds763/ProGeM
- name: GenomicRanges
  role: GRanges distance-to-TSS operations for nearest-gene selection and overlap
    of LD ranges with the gene model
- name: data.table
  role: Loading and filtering the sentinel, proxy and VEP variant tables
- name: GenomicFeatures
  role: Bioconductor dependency used to build/work with the GRCh37 gene model from
    the Ensembl GTF
- name: biomaRt
  role: Bioconductor dependency for gene annotation/biotype lookup within the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:b48ccb1aacdd831181ae0a95fad06653a82313f668560b830407770af5b0107c
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
  - data.table
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# sentinel-proxy LD variant filtering for QTL candidate gene prioritization

## Summary

ProGeM's bottom-up variant-level filtering step: filter proxy variants by an r2 threshold (default 0.8), extract the three nearest protein-coding genes to each sentinel within +/-500 kb via GenomicRanges, and flag genes overlapping the sentinel-proxy LD range to build a bottom-up candidate gene set for molecular QTLs.

## When to use

Use when you have sentinel variants plus LD proxies and GRCh37 gene coordinates and need positional bottom-up candidate causal genes for molecular QTL prioritization.

## When NOT to use

- The QTL is a trans-pQTL: the paper notes 'the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy to the one adopted here', so local positional filtering around a sentinel is not appropriate
- Input variants lack GRCh37 coordinates or LD proxy information — the nearest-gene and LD-overlap logic depends on GRCh37 sentinel/proxy start/end positions and an r2 column
- You need only top-down trait-annotation candidates without any positional/variant-level evidence — this skill is specifically the bottom-up component

## Inputs

- Tab-separated sentinel variant .txt file with columns rsID, CHR, START, END (GRCh37 coordinates; use chr:start notation if no rsID)
- Tab-separated proxy variant .txt file with columns PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2 (proxies from own sample or 1000 Genomes)
- Variant Effect Predictor (VEP) output .txt for sentinels and proxies, with one record per sentinel-proxy combination and column indices for sentinel/proxy rsIDs specified in ProGeM_settings.R
- GRCh37_genes.RData gene model built from Homo_sapiens.GRCh37.82.gtf (Ensembl)
- ProGeM_settings.R parameter file (r2 threshold, window size, biotype, gene count)

## Outputs

- OUTPUT_bottom_up_summary.txt (per-gene boolean annotations: nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL; IMPACT coded 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH)
- OUTPUT_nearest_genes.txt (genes nearest to each sentinel, filtered on user-specified biotype)
- OUTPUT_VEP_IMPACT_annotations.txt

## How to apply

Load the sentinel table (rsID, CHR, START, END), the proxy table (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2), and the GRCh37_genes.RData gene model derived from Homo_sapiens.GRCh37.82.gtf using data.table and GenomicRanges. Filter proxies at the user-defined r2 threshold (default 0.8), then compute distance-to-TSS to extract the three nearest genes per sentinel (user-adjustable count) within a +/-500 kb window, filtering on protein-coding biotype. Overlap the LD range — defined as left-most to right-most sentinel/proxy variant plus a 5 kb overhang — with the gene model to flag LD_overlapping candidates. Annotate candidates with VEP IMPACT (HIGH/MODERATE vs LOW/MODIFIER) from the user-supplied VEP output, writing OUTPUT_bottom_up_summary.txt and OUTPUT_nearest_genes.txt. Genomic proximity is grounded in the paper's finding that 69% of causal candidates were nearest to the sentinel, making proximity the most reliable single indicator.

## Related tools

- **ProGeM** (R script implementing the sentinel/proxy filtering, nearest-gene extraction and LD-range overlap to produce the bottom-up summary) — https://github.com/ds763/ProGeM
- **GenomicRanges** (GRanges distance-to-TSS operations for nearest-gene selection and overlap of LD ranges with the gene model)
- **data.table** (Loading and filtering the sentinel, proxy and VEP variant tables)
- **GenomicFeatures** (Bioconductor dependency used to build/work with the GRCh37 gene model from the Ensembl GTF)
- **biomaRt** (Bioconductor dependency for gene annotation/biotype lookup within the pipeline)

## Examples

```
source("ProGeM_settings.R"); source("ProGeM_functions.R"); source("ProGeM_commands.R")
```

## Evaluation signals

- Proxies in the working set all satisfy the configured r2 threshold (default r2 >= 0.8)
- OUTPUT_bottom_up_summary.txt contains exactly the first three nearest genes per sentinel (columns nearest, second_nearest, third_nearest are mutually exclusive booleans per row) and all rows are protein-coding biotype under the default settings
- LD_overlapping flags match genes falling within the sentinel-proxy span plus the 5 kb overhang
- lead_IMPACT/proxy_IMPACT values are restricted to {0,1,2} mapping MODIFIER/LOW, MODERATE, HIGH from the VEP IMPACT column
- Validation against the 227 mQTL reference set reproduces the reported 98% recovery of expert-curated causal genes (ProGeM missed only 4 of 227)

## Limitations

- Cis-eQTL evidence is noisy: 'cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment', so lead/proxy_eQTL flags alone should not drive prioritization
- Reference causal gene assignments may be biased toward nearest genes, and ProGeM shares KEGG with the curation process, so observed mQTL sensitivity may be inflated
- Whether genomic proximity is a good true-positive indicator for other molecular QTLs and complex traits requires further research; defaults (3 nearest genes, +/-500 kb, protein-coding only, 5 kb LD overhang) are configurable and changing them alters recovery

## Evidence

- [methods] Proxy filtering by r2 threshold: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [methods] Nearest genes selection: "the bottom-up summary will only annotate the first three"
- [methods] Gene biotype filtering: "contains basic annotation of genes nearest to each sentinel variant, filtered on the basis of user-specified biotype(s) of interest"
- [abstract] Proximity is the strongest indicator: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator of 'true positive' causal genes"
- [readme] LD region definition: "This is defined as the range between the left-most and right-most sentinel/proxy variant plus an overhang.  The user can specify the size of the overhang (default is 5kb)."
- [readme] Proxy input format: "A tab-separated .txt file containing rsIDs, chromosomes, and GRCh37 coordinates (both start and end) of proxies for your sentinel variants; along with the corresponding sentinel rsIDs and r2 value"
