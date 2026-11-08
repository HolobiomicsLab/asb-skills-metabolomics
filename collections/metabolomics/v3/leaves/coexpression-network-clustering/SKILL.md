---
name: coexpression-network-clustering
description: Use when integrating paired transcriptomic–metabolomic data to group
  co-abundant mass features and transcripts into overlapping clusters (FCs), e.g.
  to capture metabolites correlated with biosynthetic genes prior to pathway reconstruction.
when_to_use_negative:
- The analysis goal is reaction-rules-based pathway reconstruction (e.g. mapping mass
  shifts via RetroRules/LOTUS) rather than statistical clustering of correlated features
- The dataset lacks paired transcriptomic and metabolomic samples, so transcript–mass
  feature correlations cannot be computed
- Only MS/MS spectral annotation of mass features is needed — MEANtools does not currently
  support loading MS/MS data or spectral library/networking-based annotation
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: MEANtools
  role: Implements the MR-based correlation conversion, exponential decay edge weighting,
    MAD rescaling, and merge_clusters.py for merging overlapping FCs
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters the nodes and exponential-decay edge weights of each network into
    multiple overlapping feature clusters (FCs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:2cbf237e81e705b58febc92d5497818e92a688ac8cd37be3b7e862a392a8c7d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# coexpression-network-clustering

## Summary

Convert pairwise transcript–mass feature Pearson correlations into mutual ranks (MR), transform MRs into edge weights via an exponential decay function, and cluster the weighted network with ClusterONE to derive overlapping functional clusters (FCs) that group correlated metabolites and biosynthetic genes.

## When to use

Use when integrating paired transcriptomic–metabolomic data to group co-abundant mass features and transcripts into overlapping clusters (FCs), e.g. to capture metabolites correlated with biosynthetic genes prior to pathway reconstruction.

## When NOT to use

- The analysis goal is reaction-rules-based pathway reconstruction (e.g. mapping mass shifts via RetroRules/LOTUS) rather than statistical clustering of correlated features
- The dataset lacks paired transcriptomic and metabolomic samples, so transcript–mass feature correlations cannot be computed
- Only MS/MS spectral annotation of mass features is needed — MEANtools does not currently support loading MS/MS data or spectral library/networking-based annotation

## Inputs

- Paired transcriptomic–metabolomic abundance table (e.g. the tomato dataset from Jeon and colleagues 2020)
- Pairwise Pearson correlation values between transcripts and mass features
- Exponential decay parameter DR (tested at 5, 10, 25, 50)

## Outputs

- Edge-weight files for each DR network (DR = 5, 10, 25, 50)
- Overlapping feature clusters (FCs) from ClusterONE for each network
- Per-network FC membership table listing transcripts and mass features assigned to each FC (stored in SQLite)

## How to apply

Load a paired transcriptomic–metabolomic dataset (e.g. the tomato dataset of Jeon and colleagues, 2020) and compute Pearson correlations between all transcripts and mass features. Convert the pairwise Pearson values to mutual ranks (MR) using the MEANtools MR-based correlation approach, which mitigates the false-positive associations and thresholding difficulty inherent to raw Pearson-based integration. Subject the continuous MR values to an exponential decay function to obtain discrete edge weights, generating networks at decay parameters DR = 5, 10, 25, and 50, and save each network's edge-weight file. Feed the nodes and edge weights of each network into ClusterONE, which outputs multiple overlapping feature clusters (FCs); store the results in SQLite and export the per-network FC membership table. Note that the input data should first be rescaled using Median Absolute Deviation (MAD), and FCs sharing common mass features can subsequently be merged with MEANtools' merge_clusters.py.

## Related tools

- **MEANtools** (Implements the MR-based correlation conversion, exponential decay edge weighting, MAD rescaling, and merge_clusters.py for merging overlapping FCs) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters the nodes and exponential-decay edge weights of each network into multiple overlapping feature clusters (FCs))

## Evaluation signals

- FC sizes should be stable across the DR treatment combinations (DR = 5, 10, 25, 50), as reported for the Jeon and colleagues (2020) dataset (S7 Fig)
- All four DR networks should yield FCs of comparable size
- Each network's edge-weight file should be produced after MR-to-weight conversion via the exponential decay function
- ClusterONE output should contain multiple overlapping FCs, with membership stored in SQLite and exportable as a per-network FC membership table
- FCs sharing common mass features should merge cleanly via merge_clusters.py without losing feature assignments

## Limitations

- Changing ClusterONE's inbuilt size parameter alters FC size but also changes the clustering pattern of mass features and transcripts
- FC size may sometimes be insufficient for forming a complete biosynthetic pathway
- The current method to assign significance to each FC was originally developed for co-expression datasets and could be improved
- FC membership of transcripts may be affected by mass spectrometric data not capturing stereoisomers, complicating downstream pathway reconstruction

## Evidence

- [discussion] Pearson correlations to mutual ranks with exponential decay edge weights: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers"
- [discussion] ClusterONE produces overlapping FCs: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] FC size stability across DR treatments: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] MAD rescaling: "MEANtools employs the Median Absolute Deviation (MAD) for data rescaling."
- [discussion] merge_clusters.py merges FCs sharing mass features: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] ClusterONE parameter trade-off: "Changing the size of FCs is also possible using the ClusterONE [47] inbuilt parameter. However, this also changes the clustering pattern of mass features and transcripts."
- [discussion] Pearson-only integration limitations: "Integrating such datasets solely through Pearson-based correlations often results in many false-positive associations. Additionally, determining an optimal threshold for eliminating weak correlations"
- [readme] README docs pointer: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
