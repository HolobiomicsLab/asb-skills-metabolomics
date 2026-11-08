---
name: genomic-range-overlap-annotation
description: Use when annotating sentinel (and LD-proxied) variants with positional
  candidate genes from variant-level data — nearest-gene extraction, LD-range overlap,
  and biotype filtering around molecular QTL loci.
when_to_use_negative:
- 'The locus is a trans-QTL: the article notes the top-down positional/annotation
  strategy ''is not directly applicable to trans-pQTLs and would need a different
  approach'''
- Only gene-expression evidence (cis-eQTL targets) is wanted and positional proximity
  is irrelevant — the paper found cis-eQTL data yields ~3 false positives per true
  positive and is a poor standalone indicator
- The variant coordinates are not in GRCh37 — the pipeline hard-codes a GRCh37 gene
  model from Homo_sapiens.GRCh37.82.gtf and would need a lift-over first
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GenomicRanges
  role: 'GRanges interval algebra: distance-to-TSS for nearest-gene extraction and
    overlap of sentinel-proxy LD ranges against the gene model'
  repo: https://bioconductor.org/packages/release/bioc/html/GenomicRanges.html
- name: GenomicFeatures
  role: Building/loading the GRCh37 gene model from the Ensembl GTF for use as the
    overlap target
  repo: http://bioconductor.org/packages/release/bioc/html/GenomicFeatures.html
- name: data.table
  role: Fast loading and r2-based filtering (default 0.8) of the tab-separated sentinel
    and proxy variant tables
  repo: https://cran.r-project.org/web/packages/data.table/data.table.pdf
- name: ProGeM
  role: R script wrapper that executes the nearest-gene and LD-range-overlap annotations
    and writes the bottom-up output tables
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Runtime environment for the ProGeM script and Bioconductor packages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:6e0e3242df3b9aaa3d588d449172f2fba1446f8e3c39ab319ddd5ef5f49de1d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GenomicAlignments
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - biomaRt
  - data.table
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# genomic-range-overlap-annotation

## Summary

Use GenomicRanges-based interval operations over a GRCh37 gene model (built from Homo_sapiens.GRCh37.82.gtf) to annotate sentinel molecular QTL variants with candidate causal genes: the three nearest protein-coding genes by distance-to-TSS and all genes overlapping the sentinel-proxy LD range.

## When to use

Use when annotating sentinel (and LD-proxied) variants with positional candidate genes from variant-level data — nearest-gene extraction, LD-range overlap, and biotype filtering around molecular QTL loci.

## When NOT to use

- The locus is a trans-QTL: the article notes the top-down positional/annotation strategy 'is not directly applicable to trans-pQTLs and would need a different approach'
- Only gene-expression evidence (cis-eQTL targets) is wanted and positional proximity is irrelevant — the paper found cis-eQTL data yields ~3 false positives per true positive and is a poor standalone indicator
- The variant coordinates are not in GRCh37 — the pipeline hard-codes a GRCh37 gene model from Homo_sapiens.GRCh37.82.gtf and would need a lift-over first

## Inputs

- Tab-separated sentinel variant file with columns rsID | CHR | START | END in GRCh37 coordinates (rsID replaced by chr:start when absent)
- Tab-separated proxy variant file with columns PROXY_rsID | PROXY_CHR | PROXY_START | PROXY_END | LEAD_rsID | r2
- GRCh37_genes.RData gene model built from the Ensembl Homo_sapiens.GRCh37.82.gtf
- ProGeM_settings.R with user-specified parameters (nearest-gene count, local window, overhang, r2 threshold, biotype filter)

## Outputs

- OUTPUT_nearest_genes.txt — basic annotation of the three nearest genes per sentinel, filtered on user-specified biotype (default protein-coding)
- OUTPUT_bottom_up_summary.txt — per-gene boolean flags for nearest / second_nearest / third_nearest / LD_overlapping status per sentinel rsID
- LD_overlapping gene flags per sentinel-proxy pair within the GRanges gene model

## How to apply

Load the GRCh37_genes.RData gene model (derived from Homo_sapiens.GRCh37.82.gtf via GenomicFeatures/GenomicRanges) plus tab-separated sentinel (rsID, CHR, START, END) and proxy (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2) tables with data.table. First filter proxy variants at the user-defined r2 threshold (default r2 = 0.8). For each sentinel, compute distances from the variant to gene transcription start sites within the default +/-500 kb local window, rank them, and retain the three nearest genes filtered on the protein-coding biotype. Build the LD region as the range between the left-most and right-most sentinel/proxy variant plus an overhang (default 5 kb) and intersect these LD ranges with the gene model GRanges to flag LD_overlapping candidates. Rationale: genomic proximity is the most reliable single indicator of true-positive causal genes (69% of causal candidates were nearest to the sentinel), so the nearest-gene and LD-overlap annotations form the positional core of the bottom-up candidate set written to OUTPUT_nearest_genes.txt and the bottom-up summary.

## Related tools

- **GenomicRanges** (GRanges interval algebra: distance-to-TSS for nearest-gene extraction and overlap of sentinel-proxy LD ranges against the gene model) — https://bioconductor.org/packages/release/bioc/html/GenomicRanges.html
- **GenomicFeatures** (Building/loading the GRCh37 gene model from the Ensembl GTF for use as the overlap target) — http://bioconductor.org/packages/release/bioc/html/GenomicFeatures.html
- **data.table** (Fast loading and r2-based filtering (default 0.8) of the tab-separated sentinel and proxy variant tables) — https://cran.r-project.org/web/packages/data.table/data.table.pdf
- **ProGeM** (R script wrapper that executes the nearest-gene and LD-range-overlap annotations and writes the bottom-up output tables) — https://github.com/ds763/ProGeM
- **R** (Runtime environment for the ProGeM script and Bioconductor packages)

## Examples

```
Rscript -e 'source("ProGeM_settings.R"); source("ProGeM_functions.R"); source("ProGeM_commands.R")'
```

## Evaluation signals

- Per sentinel, exactly three nearest protein-coding genes (default) are flagged in columns nearest/second_nearest/third_nearest as 0/1 booleans in OUTPUT_bottom_up_summary.txt
- LD_overlapping flag is 1 only for genes whose GRanges range falls within [min(sentinel, proxy positions) - overhang, max(sentinel, proxy positions) + overhang] with default 5 kb overhang
- All candidate genes carry protein_coding gene_biotype unless the default biotype filter was explicitly disabled
- Only proxy rows with r2 >= 0.8 (default threshold) contribute to LD-range construction
- Reconstructed per-locus candidate lists reproduce the deposited ProGeM outputs (Supplementary Tables S4-S6), achieving the reported 98% recovery of the 227 expert-curated causal genes

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be biased toward nearest-gene prioritization, inflating apparent sensitivity of positional annotation
- ProGeM and the curated mQTL dataset are not fully independent (KEGG was used both by the tool and the expert review), so sensitivity may be inflated
- Whether genomic proximity indicates true causal genes for other molecular QTLs and complex traits requires further high-confidence reference datasets

## Evidence

- [methods] Filter proxy variants at the user-defined r2 threshold (default r2 = 0.8) and extract the three nearest genes to each sentinel within the default +/-500 kb local window, filtered on protein-coding biotype, using GenomicRanges distance-to-TSS operations: [evidence span withheld — anchor status: not_found]
- [readme] ProGeM will also identify all genes that overlap the so-called "LD region" surrounding each sentinel variant: "ProGeM will also identify all genes that overlap the so-called "LD region" surrounding each sentinel variant.  This is defined as the range between the left-most and right-most sentinel/proxy variant"
- [abstract] Proximity as the dominant true-positive signal: "69% of the causal candidates were nearest to the sentinel variant at the investigated molecular QTLs, indicating that genomic proximity is the most reliable indicator"
- [discussion] Configurable filtering scope: "omit the default filter on protein-coding genes only"
- [readme] ProGeM will filter based on a user-defined r2 threshold (default r2 is 0.8): "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8) if desired"
