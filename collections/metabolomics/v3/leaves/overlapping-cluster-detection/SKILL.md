---
name: overlapping-cluster-detection
description: Use when you have pairwise Pearson correlations between transcripts and
  mass features across samples and need overlapping (non-partitional) network clusters,
  since a mass feature can belong to multiple candidate biosynthetic pathways.
when_to_use_negative:
- When a strict non-overlapping partition suffices — ClusterONE's value here is that
  mass features can belong to several candidate pathway FCs simultaneously
- When inputs are raw metabolome/transcriptome tables with no cross-sample correlation
  structure computed yet; MR conversion and edge weighting require the pairwise correlation
  matrix first
- When the goal is substructure/reaction-rule pathway prediction rather than correlation-based
  clustering (that is a separate MEANtools step)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: MEANtools
  role: Converts Pearson correlations to mutual ranks, applies exponential decay edge
    weighting, invokes ClusterONE, stores FC results in SQLite, and provides merge_clusters.py
    for merging overlapping FCs
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clustering engine that takes the network nodes and edge weights and outputs
    multiple overlapping functional clusters (FCs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:a41bd2d8f5822dfacf728521b727abde13ea569218041383338d0c008ea68b96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# overlapping-cluster-detection

## Summary

Detects overlapping functional clusters (FCs) in a weighted transcript–mass feature correlation network by converting Pearson correlations to mutual ranks, applying an exponential decay edge-weight function, and running ClusterONE.

## When to use

Use when you have pairwise Pearson correlations between transcripts and mass features across samples and need overlapping (non-partitional) network clusters, since a mass feature can belong to multiple candidate biosynthetic pathways.

## When NOT to use

- When a strict non-overlapping partition suffices — ClusterONE's value here is that mass features can belong to several candidate pathway FCs simultaneously
- When inputs are raw metabolome/transcriptome tables with no cross-sample correlation structure computed yet; MR conversion and edge weighting require the pairwise correlation matrix first
- When the goal is substructure/reaction-rule pathway prediction rather than correlation-based clustering (that is a separate MEANtools step)

## Inputs

- Paired transcriptomic–metabolomic dataset (e.g. tomato dataset of Jeon and colleagues 2020)
- Pairwise Pearson correlation matrix between transcripts and mass features
- Edge-weight network files (nodes plus exponential-decay-weighted edges, one per DR = 5, 10, 25, 50)

## Outputs

- Multiple overlapping feature clusters (FCs) per network
- Per-network FC membership table listing transcripts and mass features assigned to each FC
- Clustering results stored in SQLite; merged FC sets from merge_clusters.py where FCs share mass features

## How to apply

Compute Pearson correlation values between all transcripts and mass features in the paired transcriptomic–metabolomic dataset, then convert each pairwise value to a mutual rank (MR) to dampen spurious high-correlation links. Subject the continuous MR values to an exponential decay function to obtain discrete edge weights, generating separate networks for decay parameters DR = 5, 10, 25, and 50, and save each network's edge-weight file. Exclude gene–metabolite pairs whose edge score falls below 0.01 in the Ni→j networks. Feed the nodes (transcripts and mass features) and edge weights into ClusterONE, which returns multiple overlapping FCs; store clustering results in SQLite. Optionally merge FCs sharing common mass features with the provided merge_clusters.py script, and export the per-network FC membership table.

## Related tools

- **MEANtools** (Converts Pearson correlations to mutual ranks, applies exponential decay edge weighting, invokes ClusterONE, stores FC results in SQLite, and provides merge_clusters.py for merging overlapping FCs) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clustering engine that takes the network nodes and edge weights and outputs multiple overlapping functional clusters (FCs))

## Evaluation signals

- FC sizes should be stable across the DR treatment combinations (DR = 5, 10, 25, 50), as reported for the Jeon and colleagues (2020) dataset (S7 Fig)
- All four DR networks should yield FCs of comparable size
- Gene–metabolite pairs with edge score below 0.01 must be absent from the Ni→j networks before clustering
- Output should contain overlapping clusters: some mass features appear in more than one FC
- FCs sharing common mass features should be merged by merge_clusters.py rather than kept as redundant separate clusters

## Limitations

- Changing the size of FCs via ClusterONE's inbuilt parameter also changes the clustering pattern of mass features and transcripts
- The current method to assign significance to each FC was originally developed for co-expression datasets and could be improved
- FC size may sometimes be insufficient for forming a complete biosynthetic pathway

## Evidence

- [discussion] MR conversion and decay weighting: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function"
- [discussion] ClusterONE overlapping clustering: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] Merging overlapping FCs: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] FC size stability across DR values: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] ClusterONE parameter sensitivity: "Changing the size of FCs is also possible using the ClusterONE [47] inbuilt parameter. However, this also changes the clustering pattern of mass features and transcripts."
- [readme] Official documentation pointer: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
