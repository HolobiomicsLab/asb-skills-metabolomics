---
name: metabolic-gene-set-cross-referencing
description: Use when prioritizing candidate causal genes at molecular QTL loci by
  identifying sentinel-proximal genes already implicated in a trait-relevant molecular
  phenotype (e.g. metabolism) across curated annotation databases.
when_to_use_negative:
- The molecular phenotype is not metabolism-related and no equivalent trait-specific
  gene list (.RData) has been prepared — ProGeM requires a user-supplied gene list
  for other phenotypes (e.g. lipidomics)
- The target loci are trans-pQTLs, where ProGeM's top-down local-window strategy does
  not apply and a different prioritization approach is needed
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ProGeM
  role: R script implementing the top-down cross-referencing of local genes against
    the five metabolic gene lists and the 0-5 scoring
  repo: https://github.com/ds763/ProGeM
- name: GenomicRanges
  role: Defines the +/-500 kb local window around each sentinel and intersects it
    with the gene model and metabolic gene GRanges
- name: R
  role: Execution environment for the ProGeM pipeline and its settings file ProGeM_settings.R
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:a2dd39d4c157db2db03f9e885b1029318e26e261afa159f7254e29cb493b6837
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolic-gene-set-cross-referencing

## Summary

Cross-reference protein-coding genes within a local window around each sentinel variant against five curated metabolic gene lists (GO, KEGG, Orphanet, Reactome, MGI) to score and prioritize top-down candidate causal genes at molecular QTLs, as implemented in ProGeM's top-down component.

## When to use

Use when prioritizing candidate causal genes at molecular QTL loci by identifying sentinel-proximal genes already implicated in a trait-relevant molecular phenotype (e.g. metabolism) across curated annotation databases.

## When NOT to use

- The molecular phenotype is not metabolism-related and no equivalent trait-specific gene list (.RData) has been prepared — ProGeM requires a user-supplied gene list for other phenotypes (e.g. lipidomics)
- The target loci are trans-pQTLs, where ProGeM's top-down local-window strategy does not apply and a different prioritization approach is needed

## Inputs

- annotated_metabolic_genes.RData (list of GRanges objects of metabolic-related genes from five databases)
- GRCh37_genes.RData gene model (from Homo_sapiens.GRCh37.82.gtf, Ensembl)
- Sentinel variant table (rsID, CHR, START, END in GRCh37 coordinates)
- User parameters from ProGeM_settings.R (local window size, default +/-500 kb)

## Outputs

- OUTPUT_top_down_scores.txt (per-gene boolean annotations for GO/KEGG/MGI/Orphanet/Reactome plus a 0-5 database-count score)
- OUTPUT_cooccurring_candidates.txt (genes appearing as both bottom-up and top-down candidates per sentinel)

## How to apply

Define 'local' genes as those within +/-500 kb of each sentinel variant (user-configurable), using the GRCh37 gene model (GRCh37_genes.RData from Homo_sapiens.GRCh37.82.gtf) with GenomicRanges operations. Cross-reference these local genes against five metabolic gene lists compiled in annotated_metabolic_genes.RData: (i) all genes with a GO term containing the keyword 'metabolic', (ii) all genes in the KEGG top-level metabolism pathway hsa01100, (iii) Orphanet 'rare inborn errors of metabolism' ORPHA:68367 genes, (iv) Reactome top-level metabolism pathway R-HSA-1430728 genes, and (v) human orthologs of MGI 'abnormal metabolism' MP:0005266 mouse genes. Assign each gene a boolean per database and compute a 0-5 score equal to the count of databases highlighting that top-down candidate, writing per-gene rows (LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, GO, KEGG, MGI, orphanet, reactome, score) to OUTPUT_top_down_scores.txt. Intersect these top-down candidates with the bottom-up candidate set per sentinel to derive cooccurring 'concurrent' genes, which should be treated as primary causal candidates because they satisfy both causal assumptions.

## Related tools

- **ProGeM** (R script implementing the top-down cross-referencing of local genes against the five metabolic gene lists and the 0-5 scoring) — https://github.com/ds763/ProGeM
- **GenomicRanges** (Defines the +/-500 kb local window around each sentinel and intersects it with the gene model and metabolic gene GRanges)
- **R** (Execution environment for the ProGeM pipeline and its settings file ProGeM_settings.R) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R"); source("ProGeM_functions.R"); source("ProGeM_commands.R")  # sources settings after user sets local window (default +/-500kb) and loads annotated_metabolic_genes.RData
```

## Evaluation signals

- Every top-down score must equal the sum of the five boolean database columns (columns 5-9), i.e. an integer in 0-5
- Each top-down candidate must lie within the user-defined local window (default +/-500 kb) of its sentinel in GRCh37 coordinates
- Every gene in OUTPUT_cooccurring_candidates.txt must appear in both OUTPUT_bottom_up_summary.txt and OUTPUT_top_down_scores.txt for the same sentinel
- Benchmark against the 227 curated mQTL reference set: expect ~98% recovery of expert-curated causal genes (only 4 of 227 missed)

## Limitations

- ProGeM and the curated mQTL reference dataset are not fully independent (KEGG was used both by ProGeM and to guide the expert review), so observed sensitivity at mQTLs may be inflated
- Reference causal gene assignments may be biased, e.g. toward nearest genes, which flatters proximity-driven components
- The metabolic gene lists are trait-specific; applying this skill to other molecular phenotypes requires the user to compile a new .RData gene list

## Evidence

- [readme] ProGeM has been designed to identify top-down candidates by pulling out genes that (i) reside "locally" to a sentinel variant AND (ii) have previously been implicated in the broad molecular phenotype in question.: "ProGeM has been designed to identify top-down candidates by pulling out genes that (i) reside "locally" to a sentinel variant AND (ii) have previously been implicated in the broad molecular phenotype"
- [methods] the user can specify how "locally" is defined (the default is +/-500kb): "the user can specify how "locally" is defined (the default is +/-500kb)"
- [methods] Gene Ontology (GO): all genes associated with a GO term containing the keyword "metabolic": "Gene Ontology (GO): all genes associated with a GO term containing the keyword "metabolic""
- [methods] column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator: "column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator"
- [abstract] For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
