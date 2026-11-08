---
name: metabolic-gene-set-scoring
description: Use when prioritizing candidate causal genes at a molecular QTL (mQTL)
  by testing whether genes local to the sentinel variant are annotated as metabolism-related
  in GO, KEGG (hsa01100), Orphanet (ORPHA:68367), Reactome (R-HSA-1430728) or MGI
  (MP:0005266).
when_to_use_negative:
- 'The QTL of interest is a trans-pQTL: ProGeM''s local-window top-down strategy is
  not directly applicable and would call for a different approach'
- The trait is not metabolism-related and no analogous phenotype-specific .RData gene
  list has been prepared — the default metabolic gene set would produce irrelevant
  top-down scores
- Non-protein-coding candidates matter and the default protein-coding-only biotype
  filter is retained
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_1775
- http://edamontology.org/topic_3407
tools:
- name: ProGeM
  role: R script that identifies local top-down candidates and scores them against
    the five metabolic gene databases
  repo: https://github.com/ds763/ProGeM
- name: R
  role: execution environment; ProGeM_settings.R is sourced to run the pipeline end
    to end
- name: GenomicRanges
  role: range operations to test overlap of local genes with the GRanges metabolic-gene
    sets within the ±500 kb window
- name: data.table
  role: R dependency used to read and manipulate input and output tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:d63bfe0dafd98fca01f51ce802ed2bb373393f0d66dd5de9154102e9fcae352d
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
  - data.table
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_1775
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# metabolic-gene-set-scoring

## Summary

ProGeM's top-down candidate scoring: local genes (default ±500 kb of a sentinel variant) are annotated against five metabolic-gene databases (GO, KEGG, Orphanet, Reactome, MGI) and assigned a 0–5 metabolic-relevance score, producing the OUTPUT_top_down_scores.txt table used to flag genes previously implicated in metabolism.

## When to use

Use when prioritizing candidate causal genes at a molecular QTL (mQTL) by testing whether genes local to the sentinel variant are annotated as metabolism-related in GO, KEGG (hsa01100), Orphanet (ORPHA:68367), Reactome (R-HSA-1430728) or MGI (MP:0005266).

## When NOT to use

- The QTL of interest is a trans-pQTL: ProGeM's local-window top-down strategy is not directly applicable and would call for a different approach
- The trait is not metabolism-related and no analogous phenotype-specific .RData gene list has been prepared — the default metabolic gene set would produce irrelevant top-down scores
- Non-protein-coding candidates matter and the default protein-coding-only biotype filter is retained

## Inputs

- annotated_metabolic_genes.RData (GRanges objects of metabolic genes from GO, KEGG, Orphanet, Reactome, MGI)
- Sentinel variant tab-separated .txt (rsID, CHR, START, END; GRCh37 coordinates)
- GRCh37_genes.RData gene model (from Homo_sapiens.GRCh37.82.gtf)
- ProGeM_settings.R with user-specified parameters (biotype, ±500 kb local window, tissues)

## Outputs

- OUTPUT_top_down_scores.txt (LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, GO, KEGG, MGI, orphanet, reactome, score columns; score 0–5)
- Cooccurring candidate set (genes both bottom-up AND top-down) written to OUTPUT_cooccurring_candidates.txt

## How to apply

Load the annotated_metabolic_genes.RData file, a list of GRanges objects of metabolism-related genes built from five databases: all genes with a GO term containing 'metabolic', all genes in the KEGG top-level metabolism pathway hsa01100, Orphanet 'rare inborn errors of metabolism' (ORPHA:68367) genes, Reactome top-level metabolism pathway R-HSA-1430728 genes, and human orthologs of MGI 'abnormal metabolism' (MP:0005266) mouse genes. Restrict candidates to protein-coding genes (default biotype filter) lying locally to the sentinel variant, where 'locally' defaults to ±500 kb but is user-configurable (e.g. extendable to ±1 Mb). For each local gene, set binary indicator columns 5–9 for membership in each database and compute column 10 as the sum, i.e. how many databases [out of 5] highlighted that top-down candidate mediator; e.g. MTHFR at rs1801133 scores 4 (GO, KEGG, orphanet, reactome = 1; MGI = 0). Write results to OUTPUT_top_down_scores.txt and intersect with bottom-up candidates to derive cooccurring candidates. To adapt to another phenotype (e.g. lipidomics), prepare an analogous .RData file of phenotype-relevant genes in place of the metabolic set.

## Related tools

- **ProGeM** (R script that identifies local top-down candidates and scores them against the five metabolic gene databases) — https://github.com/ds763/ProGeM
- **R** (execution environment; ProGeM_settings.R is sourced to run the pipeline end to end)
- **GenomicRanges** (range operations to test overlap of local genes with the GRanges metabolic-gene sets within the ±500 kb window)
- **data.table** (R dependency used to read and manipulate input and output tables)

## Examples

```
source("ProGeM_settings.R")  # specify biotype, +/-500kb local window, metabolic .RData; writes OUTPUT_top_down_scores.txt
```

## Evaluation signals

- Column 10 (score) of OUTPUT_top_down_scores.txt equals the row-wise sum of columns 5–9 (GO, KEGG, MGI, orphanet, reactome boolean indicators), bounded 0–5
- MTHFR at rs1801133 should appear with score 4 (GO=1, KEGG=1, MGI=0, orphanet=1, reactome=1)
- All reported top-down genes lie within the user-specified local window (default ±500 kb) of the sentinel and pass the protein-coding biotype filter
- Each of the five database indicator columns is consistent with membership in the corresponding source list (e.g. KEGG column reflects hsa01100 annotation only)

## Limitations

- KEGG was used both by ProGeM and as a guide for the expert literature review, so observed sensitivity at mQTLs may be inflated
- The metabolic gene sets are phenotype-specific; other molecular phenotypes require the user to prepare a custom .RData gene list
- cis-eQTL evidence overall led to three false positives per true positive, so top-down annotation alone is insufficient without cooccurrence with bottom-up evidence

## Evidence

- [methods] top-down scoring against five metabolic databases: "column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator"
- [readme] five database definitions: "Gene Ontology (GO): all genes associated with a GO term containing the keyword "metabolic""
- [readme] KEGG top-level pathway: "Kyoto Encyclopedia of Genes and Genomes (KEGG): all genes annotated to the top-level metabolism pathway (hsa01100)"
- [readme] top-down definition and local window: "identify top-down candidates by pulling out genes that (i) reside "locally" to a sentinel variant AND (ii) have previously been implicated"
- [readme] MTHFR example score: "rs1801133 |	ENSG00000177000 |	MTHFR |	protein_coding |	1 |	1 |	0 |	1 |	1 |	4"
- [discussion] KEGG circularity caveat: "the KEGG database was utilized both by ProGeM and as a guide for the expert literature review"
