---
name: transcript-massfeature-correlation-network-construction
description: Use when you have a paired transcriptomic–metabolomic dataset (e.g.,
  the Jeon 2020 tomato dataset) and need to derive statistically integrated, overlapping
  functional clusters linking co-abundant transcripts and mass features.
when_to_use_negative:
- Transcript and metabolite samples are not paired across the same conditions, so
  Pearson/MR correlation across samples cannot be computed
- The goal is reaction-rules-based pathway prediction (RetroRules/LOTUS adduct matching)
  rather than statistical transcript–metabolite co-abundance clustering
- A clustering result already exists and the task is pathway-level significance assignment,
  which ClusterONE's co-expression-derived statistics do not address
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3308
tools:
- name: MEANtools
  role: Implements the MR-based correlation approach, exponential decay weighting,
    edge-score filtering, and merge_clusters.py for merging overlapping FCs
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters the network nodes (transcripts and mass features) and edge weights
    to produce multiple overlapping feature clusters (FCs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:ab0614e4dfd5d679bca76a7565115982644e520f920bc326da8d36bc6214e13a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# transcript-massfeature-correlation-network-construction

## Summary

Constructs edge-weighted transcript–mass feature networks from paired transcriptomic–metabolomic data by converting Pearson correlations to mutual ranks (MRs), applying an exponential decay function with adjustable decay parameter DR, and clustering nodes with ClusterONE to obtain overlapping feature clusters (FCs).

## When to use

Use when you have a paired transcriptomic–metabolomic dataset (e.g., the Jeon 2020 tomato dataset) and need to derive statistically integrated, overlapping functional clusters linking co-abundant transcripts and mass features.

## When NOT to use

- Transcript and metabolite samples are not paired across the same conditions, so Pearson/MR correlation across samples cannot be computed
- The goal is reaction-rules-based pathway prediction (RetroRules/LOTUS adduct matching) rather than statistical transcript–metabolite co-abundance clustering
- A clustering result already exists and the task is pathway-level significance assignment, which ClusterONE's co-expression-derived statistics do not address

## Inputs

- Paired transcriptomic–metabolomic abundance dataset (e.g., tomato, Jeon and colleagues 2020) with samples matched across omics layers
- Pearson correlation matrix between transcripts and mass features
- Decay parameter DR values (5, 10, 25, 50)
- Edge score threshold (0.01)

## Outputs

- Edge-weight files for each DR network (DR = 5, 10, 25, 50)
- Overlapping feature clusters (FCs) from ClusterONE, stored in SQLite
- Per-network FC membership table listing transcripts and mass features assigned to each FC
- Merged FC sets (via merge_clusters.py) for FCs sharing common mass features

## How to apply

Load the paired transcript–metabolite abundance table (e.g., the tomato dataset from Jeon and colleagues 2020) and compute Pearson correlation values between every transcript and mass feature. Convert pairwise correlations to mutual ranks following MEANtools' MR-based correlation approach, since Pearson-based integration alone yields many false-positive associations and thresholding is difficult. Subject the continuous MR values to an exponential decay function to obtain discrete edge weights, generating networks at decay parameters DR = 5, 10, 25, and 50, excluding gene–metabolite pairs with edge score below 0.01, and saving each network's edge-weight file. Run ClusterONE on the nodes and edge weights of each network to produce multiple overlapping feature clusters (FCs), storing the results in SQLite. Use MEANtools' merge_clusters.py to merge FCs sharing common mass features and export the per-network FC membership table. Validate by confirming FC sizes are stable across all four DR treatments.

## Related tools

- **MEANtools** (Implements the MR-based correlation approach, exponential decay weighting, edge-score filtering, and merge_clusters.py for merging overlapping FCs) — github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters the network nodes (transcripts and mass features) and edge weights to produce multiple overlapping feature clusters (FCs))

## Evaluation signals

- All gene–metabolite pairs with edge score below 0.01 are excluded from the resulting networks
- FC sizes are stable (comparable) across all four DR treatment combinations (DR = 5, 10, 25, 50), as reported for the Jeon 2020 dataset (S7 Fig)
- Each of the four DR networks yields FCs of comparable size and consistent membership of transcripts and mass features
- FCs sharing common mass features are merged (merge_clusters.py) rather than left duplicated in the membership table
- Clustering results for nodes and edge weights are persisted to SQLite for downstream export

## Limitations

- Changing ClusterONE's inbuilt parameter to alter FC size also changes the clustering pattern of mass features and transcripts, so FC size tuning is entangled with cluster composition
- The current method to assign significance to each FC was originally developed for co-expression datasets and could be improved
- Integrating datasets solely through Pearson-based correlations often results in many false-positive associations, which motivates the MR transform but does not eliminate all spurious links
- FC size may sometimes be insufficient for forming a complete biosynthetic pathway

## Evidence

- [discussion] Convert pairwise transcript-mass feature Pearson correlations to mutual ranks and exponential-decay edge weights: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers"
- [discussion] ClusterONE produces overlapping FCs: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] Edge score cutoff of 0.01: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
- [discussion] FC size stability across DR treatments: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] Merging overlapping FCs: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] MR rationale against Pearson false positives: "Integrating such datasets solely through Pearson-based correlations often results in many false-positive associations."
- [readme] Documentation location for MEANtools usage: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
