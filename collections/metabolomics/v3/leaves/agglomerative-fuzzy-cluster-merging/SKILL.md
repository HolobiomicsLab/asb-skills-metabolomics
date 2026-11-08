---
name: agglomerative-fuzzy-cluster-merging
description: Use when a Fisher's-test pathway enrichment table contains many significantly
  enriched pathways with substantial content overlap (within or across pathway databases),
  and a non-redundant grouping of enriched pathways is needed for interpretation.
when_to_use_negative:
- The enrichment results contain few significant pathways with little overlap, so
  clustering would add a redundant layer over already-interpretable results
- You need standard statistical clustering (e.g., k-means or hierarchical clustering
  on expression data) rather than heuristic grouping by pathway content overlap
- You have not yet run Fisher's enrichment testing — findCluster expects the filtered
  fishers_df, not raw analyte lists
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: R package providing findCluster, FilterFishersResults, and compute_overlap_matrix
    used to cluster enriched pathways by analyte overlap
  repo: https://github.com/Mathelab/RaMP-DB
- name: R
  role: Language in which the RaMP clustering and enrichment workflow is implemented
    and run
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Backend relational database supplying the pathway-to-analyte memberships used
    to compute the Jaccard overlap matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:39ee49682e24c53a5f5c27d5b5220eeb4e6785cd7ffc1d177e11038e30d7dcf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - MySQL
  - R
  - RaMP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# agglomerative-fuzzy-cluster-merging

## Summary

A DAVID-style heuristic fuzzy multiple-linkage partitioning procedure implemented in RaMP's findCluster that groups enriched pathways by pairwise Jaccard analyte overlap, then iteratively agglomeratively merges clusters whose overlap exceeds a user-defined threshold, reducing redundancy from overlapping pathways across KEGG, Reactome, WikiPathways, and HMDB.

## When to use

Use when a Fisher's-test pathway enrichment table contains many significantly enriched pathways with substantial content overlap (within or across pathway databases), and a non-redundant grouping of enriched pathways is needed for interpretation.

## When NOT to use

- The enrichment results contain few significant pathways with little overlap, so clustering would add a redundant layer over already-interpretable results
- You need standard statistical clustering (e.g., k-means or hierarchical clustering on expression data) rather than heuristic grouping by pathway content overlap
- You have not yet run Fisher's enrichment testing — findCluster expects the filtered fishers_df, not raw analyte lists

## Inputs

- fishers_df: data frame of Fisher's-test pathway enrichment results (pathway IDs, p-values) from runFisherTest or runCombinedFisherTest
- FilterFishersResults output: table of significantly enriched pathways passing p_holmadj_cutoff and p_fdradj_cutoff
- Pathway-to-analyte membership table (pathwaysWithAnalytes: genes/metabolites per pathway from the RaMP MySQL/SQLite database)
- User-defined thresholds: perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap

## Outputs

- pathway_clusters.csv: table mapping each significant pathway ID to its cluster label
- Pairwise pathway overlap (Jaccard) matrix
- Cluster seed assignments and merged cluster membership

## How to apply

Start from the Fisher's-test results table (fishers_df) filtered to significant pathways via FilterFishersResults using user-defined Holm-adjusted (p_holmadj_cutoff) and FDR-adjusted (p_fdradj_cutoff) p-value cutoffs. Build a pairwise overlap matrix with compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod), computing Jaccard similarity of gene/metabolite memberships for every pathway pair. Identify seed pathways: mark pairs as linked when Jaccard analyte overlap exceeds perc_analyte_overlap, and promote pathways linked to at least min_pathway_tocluster other pathways to seed status, forming initial clusters; remaining pathways join the seed cluster with which they exceed the overlap threshold. Then agglomeratively merge clusters in iterative passes, joining any two clusters whose pairwise cluster-level overlap reaches perc_pathway_overlap, until no further merges occur, and write the final pathway-to-cluster assignment (pathway ID plus cluster label) to pathway_clusters.csv. The fuzzy multiple-linkage design means a pathway can be linked to several seeds, so thresholds perc_analyte_overlap, min_pathway_tocluster, and perc_pathway_overlap jointly control cluster granularity and must be tuned to the enrichment result set.

## Related tools

- **RaMP** (R package providing findCluster, FilterFishersResults, and compute_overlap_matrix used to cluster enriched pathways by analyte overlap) — https://github.com/Mathelab/RaMP-DB
- **R** (Language in which the RaMP clustering and enrichment workflow is implemented and run) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Backend relational database supplying the pathway-to-analyte memberships used to compute the Jaccard overlap matrix)

## Examples

```
findCluster(fishers_df, perc_analyte_overlap=0.3, min_pathway_tocluster=3, perc_pathway_overlap=0.5)
```

## Evaluation signals

- Every filtered significant pathway appears exactly once in pathway_clusters.csv with a non-empty cluster label
- No two clusters were merged whose pairwise overlap was below the perc_pathway_overlap threshold
- Each seed pathway is linked to at least min_pathway_tocluster other pathways with Jaccard overlap above perc_analyte_overlap
- Row count of pathway_clusters.csv equals the number of rows surviving FilterFishersResults cutoffs
- Cluster-level merge passes terminate — the iterative merging converges with no further merges possible

## Limitations

- The algorithm is heuristic and DAVID-style; results depend on user-chosen thresholds (perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap) with no automated cutoff selection
- The Mathelab/RaMP-DB repository is no longer maintained; users are directed to ncats/RaMP-DB (RaMP-DB 3.0) for updated backend data
- Overlap is computed only from analyte (gene/metabolite) membership Jaccard similarity, so pathways sharing few members but similar biology will not be clustered together

## Evidence

- [methods] findCluster(fishers_df, perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap): "findCluster(fishers_df, perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap)"
- [methods] FilterFishersResults: "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [methods] compute_overlap_matrix: "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [readme] pathway databases overlap: [evidence span withheld — anchor status: external_doc]
- [readme] repository unmaintained: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
