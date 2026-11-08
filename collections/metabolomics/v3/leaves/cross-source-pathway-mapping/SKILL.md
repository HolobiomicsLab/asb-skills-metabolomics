---
name: cross-source-pathway-mapping
description: Use when you need to measure how much metabolite content is shared between
  pathway definitions from different databases (e.g., KEGG vs Reactome vs WikiPathways)
  to assess redundancy before enrichment analysis or pathway clustering.
when_to_use_negative:
- Input is a metabolite list requiring pathway enrichment (use RaMP's runFisherTest/runCombinedFisherTest
  instead, not overlap matrices)
- The analysis needs gene/protein pathway overlap rather than metabolite-only overlap
  (the 'together' parameter would need combined analytes, not metabolites-only)
- You need a single pathway's membership query rather than cross-source redundancy
  assessment (use rampFastPathFromSource or getPathwayFromAnalyte)
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: RaMP
  role: Provides connectToRaMP, compute_overlap_matrix, and updateOverlapMatrix functions
    used to connect to the database, compute Jaccard metabolite overlap between pathway
    pairs, and assemble the full overlap matrix
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R database interface used by RaMP's connectToRaMP to query the locally built
    MySQL RaMP database for pathway-metabolite memberships
- name: RaMP-BackEnd
  role: Python/C# scripts used to create the RaMP MySQL database from HMDB, KEGG,
    Reactome, and WikiPathways source files before overlap computation
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:3eaabeb53653b320522ca58ea85cbea27c7409d91a5ccd6186157c4f23782cbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - DBI
  - R
  - RMySQL
  - RaMP
  - RaMP-BackEnd
  - dplyr
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-source-pathway-mapping

## Summary

Quantifies pairwise metabolite-content overlap between pathway definitions drawn from different source databases (KEGG, Reactome, WikiPathways) integrated in RaMP, using the Jaccard index to expose pathway redundancy across sources.

## When to use

Use when you need to measure how much metabolite content is shared between pathway definitions from different databases (e.g., KEGG vs Reactome vs WikiPathways) to assess redundancy before enrichment analysis or pathway clustering.

## When NOT to use

- Input is a metabolite list requiring pathway enrichment (use RaMP's runFisherTest/runCombinedFisherTest instead, not overlap matrices)
- The analysis needs gene/protein pathway overlap rather than metabolite-only overlap (the 'together' parameter would need combined analytes, not metabolites-only)
- You need a single pathway's membership query rather than cross-source redundancy assessment (use rampFastPathFromSource or getPathwayFromAnalyte)

## Inputs

- Locally built RaMP MySQL database (mysql dump from RaMP-DB inst/extdata or built via Mathelab/RaMP-BackEnd)
- Pathway-to-metabolite membership tables for KEGG, Reactome, and WikiPathways pathways
- Database connection parameters (dbname, username, conpass, host)

## Outputs

- Pairwise metabolite Jaccard overlap matrix restricted to KEGG-Reactome-WikiPathways pathway comparisons (e.g., metabolite_overlap_matrix.csv)
- Identification of which database pairs exhibit the largest metabolite overlap

## How to apply

Connect to a locally built RaMP MySQL database with connectToRaMP(dbname, username, conpass, host) via RMySQL/DBI, then retrieve the pathway-to-metabolite membership tables for all pathways annotated to KEGG, Reactome, and WikiPathways. For every pathway pair, compute the metabolite Jaccard index (shared metabolites divided by union size) with compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod), setting overlapmethod to the Jaccard metric. Assemble the per-pair scores into full pairwise overlap matrices using updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username), restricting the analyte set to metabolites only (together set to metabolites-only rather than combined gene-plus-metabolite analytes). Finally, subset the matrix to KEGG-Reactome-WikiPathways pathway comparisons and export it (e.g., metabolite_overlap_matrix.csv); inspect the block structure to identify which database pairs show the largest overlap — in the source analysis, pathways within Reactome and KEGG showed the largest number of overlapping pathways.

## Related tools

- **RaMP** (Provides connectToRaMP, compute_overlap_matrix, and updateOverlapMatrix functions used to connect to the database, compute Jaccard metabolite overlap between pathway pairs, and assemble the full overlap matrix) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R database interface used by RaMP's connectToRaMP to query the locally built MySQL RaMP database for pathway-metabolite memberships)
- **RaMP-BackEnd** (Python/C# scripts used to create the RaMP MySQL database from HMDB, KEGG, Reactome, and WikiPathways source files before overlap computation) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
con <- RaMP::connectToRaMP(dbname='ramp', username='root', conpass='***', host='localhost'); om <- RaMP::updateOverlapMatrix(min_analyte=1, overlapmethod='jaccard', together=FALSE, conpass='***', host='localhost', dbname='ramp', username='root')
```

## Evaluation signals

- All pairwise Jaccard entries in the output matrix lie in [0, 1]
- The matrix is symmetric with unit diagonal (each pathway overlaps itself with Jaccard = 1)
- The cross-source block structure qualitatively reproduces the paper's redundancy pattern: pathways within Reactome and KEGG show the largest number of overlapping pathways (cf. Figure 3 of the paper)
- The exported CSV contains exactly the KEGG-Reactome-WikiPathways pathway pairs, with metabolites-only analyte restriction applied via the 'together' parameter
- The overlap computation reproduces the RaMP overlap figures also generated by the backend Venn scripts (threeVenn.R/fourVenn.R) for gene/metabolite overlaps between RaMP databases

## Limitations

- The overlap depends on the RaMP database build vintage; updates for databases in RaMP are checked multiple times a year, so overlap matrices from different RaMP versions may differ
- The original Mathelab/RaMP-DB repo is no longer maintained; users are directed to https://github.com/ncats/RaMP-DB/ where the backend and schema have changed (RaMP-DB 3.0 with expanded annotations from HMDB, KEGG, Lipid-MAPS, WikiPathways, Reactome, CheBI, PFOCR, RefMet, and Rhea)
- Building the RaMP database requires downloading content from HMDB, KEGG, Reactome, and WikiPathways via main.py with getDatabaseFiles set to True, plus a C# bulk-import step (mySQLBulkCsharp Program.cs) into MySQL — a nontrivial multi-language setup

## Evidence

- [methods] compute_overlap_matrix function signature: "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [methods] updateOverlapMatrix function signature: "updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username)"
- [abstract] RaMP aggregates four pathway databases including the three compared here: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [readme] README: RaMP is a conglomerate of four databases with pathways and annotations for genes and metabolites: [evidence span withheld — anchor status: external_doc]
- [readme] README: R Venn scripts quantify overlapping genes and metabolites between all databases in RaMP: [evidence span withheld — anchor status: external_doc]
