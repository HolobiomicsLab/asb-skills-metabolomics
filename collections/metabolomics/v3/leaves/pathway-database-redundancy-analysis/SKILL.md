---
name: pathway-database-redundancy-analysis
description: Use when you need to measure the extent of shared metabolite content
  between pathways drawn from multiple source databases (KEGG, Reactome, WikiPathways)
  hosted in a RaMP MySQL instance, e.g. to assess cross-database pathway redundancy.
when_to_use_negative:
- The goal is pathway overrepresentation/enrichment testing from a metabolite or gene
  list — use RaMP's runFisherTest / runCombinedFisherTest instead of overlap matrices.
- The analysis requires combined gene-plus-metabolite analyte overlap ('together'
  set to combined) rather than metabolites-only redundancy.
- The pathways of interest are not integrated into RaMP (e.g. pathways from a source
  outside KEGG, Reactome, WikiPathways, HMDB).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3400
tools:
- name: RaMP
  role: Provides connectToRaMP, compute_overlap_matrix, and updateOverlapMatrix functions
    used to connect to the database and build the pairwise Jaccard overlap matrix
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R driver used by connectToRaMP to connect to the RaMP MySQL database and fetch
    pathway-metabolite memberships
- name: MySQL
  role: Hosts the locally built RaMP relational database that supplies the pathway-to-metabolite
    membership tables
- name: DBI
  role: R database interface layer underlying the RaMP MySQL connection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:97d08aa13a12d0e31d8b2e79da66b8f758de35b6a23167d919fe942e7a65fce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - DBI
  - MySQL
  - R
  - RMySQL
  - RaMP
  - dplyr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3400
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pathway-database-redundancy-analysis

## Summary

Quantifies metabolite-content redundancy between biochemical pathways integrated from KEGG, Reactome, and WikiPathways in RaMP by computing pairwise Jaccard overlap matrices, identifying which database pairs contain the most overlapping pathways.

## When to use

Use when you need to measure the extent of shared metabolite content between pathways drawn from multiple source databases (KEGG, Reactome, WikiPathways) hosted in a RaMP MySQL instance, e.g. to assess cross-database pathway redundancy.

## When NOT to use

- The goal is pathway overrepresentation/enrichment testing from a metabolite or gene list — use RaMP's runFisherTest / runCombinedFisherTest instead of overlap matrices.
- The analysis requires combined gene-plus-metabolite analyte overlap ('together' set to combined) rather than metabolites-only redundancy.
- The pathways of interest are not integrated into RaMP (e.g. pathways from a source outside KEGG, Reactome, WikiPathways, HMDB).

## Inputs

- Locally built RaMP MySQL database (connection parameters: dbname, username, conpass, host)
- Pathway-to-metabolite membership tables for KEGG, Reactome, and WikiPathways pathways
- Overlap method choice (Jaccard) and min_analyte filter threshold

## Outputs

- Pairwise metabolite Jaccard overlap matrix restricted to KEGG-Reactome-WikiPathways pathway comparisons
- metabolite_overlap_matrix.csv export
- Ranked identification of which database pairs show the largest metabolite overlap

## How to apply

Connect to the locally built RaMP MySQL database with RaMP's connectToRaMP(dbname, username, conpass, host) via RMySQL/DBI, and retrieve the pathway-to-metabolite membership tables for all pathways annotated to KEGG, Reactome, and WikiPathways. For every pair of pathways, compute the metabolite Jaccard index — shared metabolites divided by the size of the union — using compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod) with overlapmethod set to the Jaccard metric. Assemble per-pair scores into full pairwise overlap matrices with updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username), setting the 'together' parameter to metabolites-only rather than combined analytes so gene memberships do not contaminate the metabolite overlap. The min_analyte threshold filters out pathways with very few metabolites before scoring. Export the matrix restricted to KEGG-Reactome-WikiPathways comparisons as metabolite_overlap_matrix.csv and inspect the cross-source block structure for redundant pathway clusters.

## Related tools

- **RaMP** (Provides connectToRaMP, compute_overlap_matrix, and updateOverlapMatrix functions used to connect to the database and build the pairwise Jaccard overlap matrix) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R driver used by connectToRaMP to connect to the RaMP MySQL database and fetch pathway-metabolite memberships)
- **MySQL** (Hosts the locally built RaMP relational database that supplies the pathway-to-metabolite membership tables)
- **DBI** (R database interface layer underlying the RaMP MySQL connection)

## Examples

```
con <- RaMP::connectToRaMP(dbname='ramp', username='user', conpass='pw', host='localhost'); M <- RaMP::updateOverlapMatrix(min_analyte=3, overlapmethod='jaccard', together=FALSE, conpass='pw', host='localhost', dbname='ramp', username='user')
```

## Evaluation signals

- All Jaccard entries in the pairwise overlap matrix fall within [0, 1].
- The matrix diagonal equals 1 (each pathway fully overlaps itself) and the matrix is symmetric.
- The cross-source block structure shows high overlap between related pathways from different databases, qualitatively reproducing the redundancy pattern shown in Figure 3 of the paper.
- Pathways within Reactome and KEGG emerge as the pairs with the largest number of overlapping pathways, matching the reported finding.
- The exported metabolite_overlap_matrix.csv is restricted to KEGG-Reactome-WikiPathways comparisons and excludes gene-only memberships (metabolites-only 'together' setting).

## Limitations

- The original Mathelab/RaMP-DB repository is no longer maintained; current RaMP versions are hosted at ncats/RaMP-DB (RaMP-DB 3.0, with expanded annotations from additional sources including ChEBI, Lipid-MAPS, PFOCR, RefMet, and Rhea), so results may differ from the 2018 database snapshot.
- Overlap scores depend on the min_analyte threshold used in updateOverlapMatrix; small pathways filtered out by this threshold are excluded from the redundancy matrix.
- The original RaMP-BackEnd build pipeline required specific tooling (python 3.5 for main.py, a C# bulk-import script) and the data/sql/output folders were too large to include in the repository.

## Evidence

- [methods] compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod): "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [methods] updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username): "updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username)"
- [abstract] RaMP combines pathways from KEGG, Reactome, WikiPathways, HMDB: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [readme] README describes RaMP as a conglomerate of four databases: [evidence span withheld — anchor status: external_doc]
- [readme] README notes the Mathelab repo is unmaintained: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
