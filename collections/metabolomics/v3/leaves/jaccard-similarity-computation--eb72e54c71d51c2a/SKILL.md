---
name: jaccard-similarity-computation
description: Use when quantifying metabolite-content overlap between pathway pairs
  — e.g., building a pairwise Jaccard redundancy matrix across KEGG, Reactome, and
  WikiPathways pathways stored in RaMP to identify which database pairs show the largest
  overlap.
when_to_use_negative:
- When the goal is pathway overrepresentation (Fisher) testing rather than pathway–pathway
  redundancy quantification — use runFisherTest/runCombinedFisherTest instead
- When overlap should be computed on combined analytes (genes and metabolites together)
  rather than metabolites-only — the 'together' parameter must not be set to metabolites-only
  in that case
- When a single analyte's pathway annotations are needed (e.g., metabolite-to-pathway
  lookup) — rampFastCata or getPathwayFromAnalyte is the appropriate query rather
  than an overlap matrix
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: RaMP
  role: Provides compute_overlap_matrix and updateOverlapMatrix functions used to
    compute per-pair Jaccard scores and assemble the full metabolite overlap matrix
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R database backend used to connect to and query the locally built RaMP MySQL
    database during overlap computation
- name: R
  role: Implementation environment for the RaMP package functions that run the Jaccard
    overlap computation
- name: MySQL
  role: Hosts the RaMP relational database of pathway-metabolite memberships that
    the overlap matrix is computed from
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:fc5fd00b1f4078b5494da352005b2ba341c569f63f24bd2e3aa751d215d6b195
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# jaccard-similarity-computation

## Summary

Computes pairwise metabolite-content overlap between pathways using the Jaccard index (shared metabolites divided by union size), assembling per-pair scores into full overlap matrices via RaMP's compute_overlap_matrix and updateOverlapMatrix functions to quantify redundancy across KEGG, Reactome, and WikiPathways.

## When to use

Use when quantifying metabolite-content overlap between pathway pairs — e.g., building a pairwise Jaccard redundancy matrix across KEGG, Reactome, and WikiPathways pathways stored in RaMP to identify which database pairs show the largest overlap.

## When NOT to use

- When the goal is pathway overrepresentation (Fisher) testing rather than pathway–pathway redundancy quantification — use runFisherTest/runCombinedFisherTest instead
- When overlap should be computed on combined analytes (genes and metabolites together) rather than metabolites-only — the 'together' parameter must not be set to metabolites-only in that case
- When a single analyte's pathway annotations are needed (e.g., metabolite-to-pathway lookup) — rampFastCata or getPathwayFromAnalyte is the appropriate query rather than an overlap matrix

## Inputs

- RaMP pathway-to-metabolite membership tables for pathways from KEGG, Reactome, and WikiPathways
- RaMP MySQL database connection parameters (dbname, username, conpass, host)
- Jaccard overlap method selection and minimum analyte threshold (min_analyte)

## Outputs

- Pairwise metabolite Jaccard overlap matrix restricted to KEGG–Reactome–WikiPathways pathway comparisons
- CSV export of the overlap matrix (e.g., metabolite_overlap_matrix.csv)

## How to apply

First connect to the locally built RaMP MySQL database with connectToRaMP(dbname, username, conpass, host) using RMySQL/DBI. Retrieve the pathway-to-metabolite membership tables for all pathways annotated to KEGG, Reactome, and WikiPathways. For every pathway pair, compute the Jaccard index — number of shared metabolites divided by the size of the union — using compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod) with overlapmethod set to the Jaccard metric. Assemble per-pair scores into full pairwise overlap matrices with updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username), setting 'together' to metabolites-only rather than combined analytes so the redundancy assessment is metabolite-specific. Export the resulting matrix restricted to KEGG–Reactome–WikiPathways comparisons (e.g., to metabolite_overlap_matrix.csv) for downstream redundancy inspection.

## Related tools

- **RaMP** (Provides compute_overlap_matrix and updateOverlapMatrix functions used to compute per-pair Jaccard scores and assemble the full metabolite overlap matrix) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R database backend used to connect to and query the locally built RaMP MySQL database during overlap computation)
- **R** (Implementation environment for the RaMP package functions that run the Jaccard overlap computation)
- **MySQL** (Hosts the RaMP relational database of pathway-metabolite memberships that the overlap matrix is computed from)

## Examples

```
con <- connectToRaMP(dbname, username, conpass, host); updateOverlapMatrix(min_analyte=1, overlapmethod='jaccard', together='metabolites', conpass, host, dbname, username)
```

## Evaluation signals

- All Jaccard entries in the overlap matrix lie in [0, 1], since the index is defined as |intersection| / |union|
- The matrix is symmetric with diagonal entries equal to 1 (each pathway perfectly overlaps itself)
- The cross-source block structure shows high Jaccard values between related pathways from different databases, qualitatively reproducing the redundancy pattern shown in Figure 3 of the paper
- Pathways within Reactome and KEGG show the largest number of overlapping pathways, consistent with the reported finding
- Matrix construction used metabolites-only ('together' set to metabolites rather than combined analytes), verified by checking that only metabolite memberships contribute to overlaps

## Limitations

- Overlap is only as current as the RaMP database build; the Mathelab/RaMP-DB repository is no longer maintained and users are directed to ncats/RaMP-DB
- High overlap between pathways from different databases indicates redundancy that must be accounted for (e.g., via clustering) before interpreting enrichment results
- The paper's database combines only KEGG, Reactome, WikiPathways, and HMDB in the 2018 version, so overlap conclusions are limited to those sources

## Evidence

- [methods] compute_overlap_matrix: "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [methods] updateOverlapMatrix: "updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username)"
- [abstract] RaMP scope: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways"
- [readme] Database composition: [evidence span withheld — anchor status: external_doc]
