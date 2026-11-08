---
name: pathway-overrepresentation-clustering
description: Use when a Fisher's enrichment result table (fishers_df) contains many
  overlapping significant pathways across KEGG, Reactome, WikiPathways, and HMDB,
  and you need to reduce redundancy into non-overlapping pathway clusters.
when_to_use_negative:
- Input is a raw analyte list rather than a filtered Fisher's enrichment results table
  — run runFisherTest and FilterFishersResults first
- The goal is building or updating the RaMP MySQL backend database itself (use Mathelab/RaMP-BackEnd
  scripts instead of findCluster)
- Only a single pathway or a small non-overlapping set of enriched pathways is under
  review, where clustering by overlap adds no value
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RaMP
  role: provides findCluster, FilterFishersResults, and compute_overlap_matrix functions
    that implement the fuzzy multiple-linkage partitioning over enriched pathways
    from KEGG, Reactome, WikiPathways, and HMDB
  repo: https://github.com/Mathelab/RaMP-DB
- name: R
  role: R package runtime in which findCluster and the clustering workflow are executed
  repo: https://github.com/Mathelab/RaMP-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:28421f5018c64065afd25646cbe46e498f483fcca0ed6880106b45bc4a704dcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - R
  - RaMP
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pathway-overrepresentation-clustering

## Summary

Groups significantly enriched pathways from a Fisher's overrepresentation test into redundant-reducing clusters using RaMP's DAVID-style heuristic fuzzy multiple-linkage partitioning: Jaccard analyte-overlap seeds, threshold-based assignment, and iterative agglomerative cluster merging.

## When to use

Use when a Fisher's enrichment result table (fishers_df) contains many overlapping significant pathways across KEGG, Reactome, WikiPathways, and HMDB, and you need to reduce redundancy into non-overlapping pathway clusters.

## When NOT to use

- Input is a raw analyte list rather than a filtered Fisher's enrichment results table — run runFisherTest and FilterFishersResults first
- The goal is building or updating the RaMP MySQL backend database itself (use Mathelab/RaMP-BackEnd scripts instead of findCluster)
- Only a single pathway or a small non-overlapping set of enriched pathways is under review, where clustering by overlap adds no value

## Inputs

- fishers_df: Fisher's test results table from runFisherTest or runCombinedFisherTest
- pathwaysWithAnalytes: pathway-to-analyte membership table used to compute Jaccard overlap
- perc_analyte_overlap: threshold for pairwise Jaccard analyte overlap between pathways
- min_pathway_tocluster: minimum number of linked pathways required to promote a pathway to seed status
- perc_pathway_overlap: threshold for merging two clusters during agglomeration

## Outputs

- pathway_clusters.csv: table with pathway ID and cluster label for every significant enriched pathway
- pairwise pathway overlap matrix (Jaccard analyte-overlap scores)

## How to apply

First run the overrepresentation test with runFisherTest (or runCombinedFisherTest for combined genes and metabolites) and retain only significant pathways via FilterFishersResults using user-defined Holm-adjusted (p_holmadj_cutoff) and FDR-adjusted (p_fdradj_cutoff) p-value cutoffs. For every pair of retained pathways, compute the pairwise Jaccard similarity of analyte (gene/metabolite) memberships with compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod) to build the overlap matrix. Identify cluster seeds by marking pairs linked when Jaccard analyte overlap exceeds perc_analyte_overlap, promoting pathways linked to at least min_pathway_tocluster other pathways to seed status. Assign remaining pathways to seeds above the threshold, then agglomeratively merge clusters in iterative passes whenever pairwise cluster-level overlap reaches perc_pathway_overlap, until no further merges occur. Write the final pathway-to-cluster assignments (pathway ID and cluster label) to pathway_clusters.csv.

## Related tools

- **RaMP** (provides findCluster, FilterFishersResults, and compute_overlap_matrix functions that implement the fuzzy multiple-linkage partitioning over enriched pathways from KEGG, Reactome, WikiPathways, and HMDB) — https://github.com/Mathelab/RaMP-DB
- **R** (R package runtime in which findCluster and the clustering workflow are executed) — https://github.com/Mathelab/RaMP-DB

## Examples

```
clusters <- findCluster(fishers_df, perc_analyte_overlap = 0.3, min_pathway_tocluster = 2, perc_pathway_overlap = 0.5)
```

## Evaluation signals

- Every filtered significant pathway appears exactly once in pathway_clusters.csv with a non-empty cluster label
- No final cluster was merged below the perc_pathway_overlap threshold
- Every cluster seed was linked to at least min_pathway_tocluster other pathways at Jaccard overlap above perc_analyte_overlap
- Pairwise Jaccard scores in the overlap matrix lie in [0, 1] and the matrix is symmetric over all retained pathway pairs
- Cluster count decreases monotonically across agglomerative merge passes until no pair exceeds perc_pathway_overlap

## Limitations

- The Mathelab/RaMP-DB repo is no longer maintained; users are directed to https://github.com/ncats/RaMP-DB/ (RaMP-DB 3.0) for continued use
- Clusters depend on user-chosen thresholds (perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap), so results vary with cutoffs
- The heuristic is DAVID-style partitioning of enriched pathways only; it does not itself re-rank pathways or recompute enrichment statistics

## Evidence

- [methods] FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff): "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [methods] findCluster(fishers_df, perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap): "findCluster(fishers_df, perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap)"
- [methods] compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod): "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [abstract] we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [readme] RaMP is a conglomerate of 4 different databases (Kegg, Reactome, HMDB, and WikiPathways), which includes pathways and annotations for genes and metabolites.: [evidence span withheld — anchor status: external_doc]
