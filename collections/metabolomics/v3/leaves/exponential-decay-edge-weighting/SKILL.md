---
name: exponential-decay-edge-weighting
description: Use when you have a paired transcriptomic-metabolomic dataset (e.g.,
  the tomato Jeon and colleagues 2020 dataset) and need to turn raw Pearson correlation
  values between transcripts and mass features into weighted edges for network clustering.
when_to_use_negative:
- The goal is reaction-rules-based pathway prediction rather than statistical correlation-based
  clustering of mass features and transcripts
- Input is a single-omics dataset (no paired transcriptomic and metabolomic samples),
  so transcript-mass feature correlations cannot be computed
- Only mass-shift or MS/MS spectral-library annotation of mass features is needed,
  with no co-abundance network intended
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MEANtools
  role: Implements the MR-based correlation approach and the exponential decay function
    that converts continuous MR values into discrete edge weights
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Consumes the decay-weighted nodes and edge weights to produce multiple overlapping
    feature clusters (FCs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:4f91153453368b5be0c53667edaebce7db7a761370c891a9c084167e5f30f302
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# exponential-decay-edge-weighting

## Summary

MEANtools' statistical-integration step that converts pairwise Pearson correlations between transcripts and mass features into mutual ranks (MRs) and then into discrete edge weights via an exponential decay function, producing weighted networks ready for overlapping ClusterONE clustering.

## When to use

Use when you have a paired transcriptomic-metabolomic dataset (e.g., the tomato Jeon and colleagues 2020 dataset) and need to turn raw Pearson correlation values between transcripts and mass features into weighted edges for network clustering.

## When NOT to use

- The goal is reaction-rules-based pathway prediction rather than statistical correlation-based clustering of mass features and transcripts
- Input is a single-omics dataset (no paired transcriptomic and metabolomic samples), so transcript-mass feature correlations cannot be computed
- Only mass-shift or MS/MS spectral-library annotation of mass features is needed, with no co-abundance network intended

## Inputs

- Paired transcriptomic-metabolomic abundance matrix (samples x transcripts and samples x mass features)
- Pairwise Pearson correlation values between transcripts and mass features
- Exponential decay parameter DR (e.g., 5, 10, 25, 50)

## Outputs

- Mutual-rank (MR) matrix of transcript-mass feature pairs
- Edge-weight files for each DR network (e.g., DR = 5, 10, 25, 50)
- Node and edge-weight files formatted for ClusterONE input
- Overlapping feature clusters (FCs) with per-network FC membership tables

## How to apply

First compute Pearson correlation values between every transcript and mass feature across samples. Convert the pairwise correlation values to mutual ranks (MR) following the MEANtools MR-based correlation approach, which mitigates the many false-positive associations that arise from using raw Pearson correlations alone. Subject the continuous MR values to an exponential decay function to obtain discrete edge weights, and generate one network per decay parameter (MEANtools demonstrates DR = 5, 10, 25, and 50), saving each network's edge-weight file. Feed both node types and edge weights into ClusterONE to produce multiple overlapping feature clusters (FCs), and optionally merge FCs sharing common mass features with merge_clusters.py. Exclude gene-metabolite pairs with edge scores below 0.01 in the Ni→j networks.

## Related tools

- **MEANtools** (Implements the MR-based correlation approach and the exponential decay function that converts continuous MR values into discrete edge weights) — github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Consumes the decay-weighted nodes and edge weights to produce multiple overlapping feature clusters (FCs))

## Evaluation signals

- FC sizes are stable across all DR treatment combinations (DR = 5, 10, 25, 50), as reported for the Jeon and colleagues (2020) dataset (S7 Fig)
- All four DR networks yield FCs of comparable size
- Edge weights are discrete values derived from continuous MRs via the exponential decay function, not raw Pearson correlations
- Gene-metabolite pairs with edge scores below 0.01 are absent from the resulting Ni→j networks
- ClusterONE outputs multiple overlapping FCs, not a disjoint partition

## Limitations

- Pearson-based correlation integration alone yields many false-positive associations and an optimal threshold for eliminating weak correlations is difficult to determine, which is why MR-based weighting is used
- Changing ClusterONE's inbuilt parameter to alter FC size also changes the clustering pattern of mass features and transcripts
- The current method to assign significance to each FC was originally developed for co-expression datasets and could be improved

## Evidence

- [discussion] correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers"
- [discussion] Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs.: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] Integrating such datasets solely through Pearson-based correlations often results in many false-positive associations.: "Integrating such datasets solely through Pearson-based correlations often results in many false-positive associations."
- [discussion] The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig).: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks.: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
