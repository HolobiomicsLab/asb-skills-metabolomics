---
name: jaccard-analyte-overlap-scoring
description: Use when a pathway enrichment result table contains many overlapping
  enriched pathways (within or across pathway databases) and you need to quantify
  and cluster their analyte-membership redundancy before interpretation.
when_to_use_negative:
- The input is an unfiltered enrichment table — apply FilterFishersResults first so
  clustering runs only on significant pathways
- You need pathway similarity based on shared function or ontology rather than shared
  analyte (gene/metabolite) membership
- The analyte list contains few pathways with little overlap, so partitioning adds
  no interpretive value over reading the enrichment table directly
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: R package providing findCluster, compute_overlap_matrix, and FilterFishersResults
    for Jaccard-based clustering of enriched pathways
  repo: https://github.com/Mathelab/RaMP-DB
- name: R
  role: Language in which the RaMP package implementing the clustering workflow is
    delivered
  repo: https://github.com/Mathelab/RaMP-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:459a16f72a84f02b7f0212972b44a105829a9e0de22458f292298fdd4ffbabf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - R
  - RaMP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# jaccard-analyte-overlap-scoring

## Summary

Computes pairwise Jaccard similarity of gene/metabolite memberships between significantly enriched pathways and uses those scores as the linkage basis for DAVID-style fuzzy multiple-linkage partitioning clustering in RaMP, resolving redundancy from pathway content overlap across KEGG, Reactome, WikiPathways, and HMDB.

## When to use

Use when a pathway enrichment result table contains many overlapping enriched pathways (within or across pathway databases) and you need to quantify and cluster their analyte-membership redundancy before interpretation.

## When NOT to use

- The input is an unfiltered enrichment table — apply FilterFishersResults first so clustering runs only on significant pathways
- You need pathway similarity based on shared function or ontology rather than shared analyte (gene/metabolite) membership
- The analyte list contains few pathways with little overlap, so partitioning adds no interpretive value over reading the enrichment table directly

## Inputs

- fishers_df: Fisher's over-representation test results for pathways (from runFisherTest or runCombinedFisherTest)
- Significant pathway list filtered by Holm-adjusted (p_holmadj_cutoff) and FDR-adjusted (p_fdradj_cutoff) p-value cutoffs
- Pathway-to-analyte membership mappings (pathwaysWithAnalytes: genes and/or metabolites per pathway)
- User-defined thresholds: perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap

## Outputs

- Pairwise Jaccard analyte-overlap matrix between enriched pathways
- pathway_clusters.csv: pathway ID and cluster label per enriched pathway

## How to apply

Start from the Fisher's test results filtered with FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff), retaining only significant pathways. For every pair of retained pathways, compute the Jaccard similarity of their analyte (gene/metabolite) memberships using compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod) to build a pairwise overlap matrix. Mark pathway pairs as linked when Jaccard analyte overlap exceeds perc_analyte_overlap, and promote pathways linked to at least min_pathway_tocluster other pathways to seed status, forming initial clusters. Then agglomeratively merge clusters in iterative passes whenever cluster-level pairwise overlap reaches perc_pathway_overlap, until no further merges occur. The Jaccard scores are the decision variable at every linkage and merge step, so threshold choices (perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap) directly control cluster granularity.

## Related tools

- **RaMP** (R package providing findCluster, compute_overlap_matrix, and FilterFishersResults for Jaccard-based clustering of enriched pathways) — https://github.com/Mathelab/RaMP-DB
- **R** (Language in which the RaMP package implementing the clustering workflow is delivered) — https://github.com/Mathelab/RaMP-DB

## Examples

```
findCluster(fishers_df, perc_analyte_overlap, min_pathway_tocluster, perc_pathway_overlap)
```

## Evaluation signals

- Every filtered significant pathway appears exactly once in pathway_clusters.csv with a non-empty cluster label
- No final cluster was merged from two clusters whose pairwise cluster-level overlap was below perc_pathway_overlap
- Seed pathways are those linked to at least min_pathway_tocluster other pathways with Jaccard analyte overlap above perc_analyte_overlap
- The overlap matrix is symmetric with unit diagonal (Jaccard of a pathway with itself = 1) and covers all retained pathway IDs

## Limitations

- The original Mathelab/RaMP-DB repo is no longer maintained; users are directed to ncats/RaMP-DB
- Clustering granularity is sensitive to the user-defined thresholds perc_analyte_overlap, min_pathway_tocluster, and perc_pathway_overlap
- The Jaccard linkage approach is heuristic (DAVID-style fuzzy multiple-linkage partitioning) and does not guarantee globally optimal clusters

## Evidence

- [methods] Jaccard analyte-overlap scoring drives seed selection and merging: "compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod)"
- [methods] Filtering precedes overlap scoring: "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [abstract] Purpose of clustering is redundancy across pathway databases: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [readme] Package maintenance status: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
