---
name: mutual-rank-correlation-analysis
description: Use when integrating paired transcriptomic–metabolomic datasets (e.g.,
  tomato samples from Jeon 2020) to find mass features correlated with biosynthetic
  genes without a prior 'bait', before clustering mass feature–transcript networks.
when_to_use_negative:
- When the goal is sequence/structure-based gene cluster identification rather than
  cross-omics correlation — use tools like plantiSMASH, PhytoClust, or PlantClusterFinder
  instead.
- When prior knowledge of a biosynthetic 'bait' enzyme or compound already anchors
  the pathway and a simple co-expression correlation would suffice.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MEANtools
  role: Implements MAD rescaling, MR-based correlation, exponential-decay edge weighting,
    and network generation from transcript–mass feature Pearson correlations
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters nodes and exponential-decay edge weights of the MR networks to produce
    multiple overlapping functional clusters (FCs)
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_006
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:0ff23fa391e556a57558824f868eaf6276729957fb4f3a920012559971a362de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# mutual-rank-correlation-analysis

## Summary

MEANtools-style integration of transcriptomic and metabolomic data: pairwise Pearson correlations between transcripts and mass features are converted to mutual ranks (MR) and then, via an exponential decay function, into discrete edge weights to build weighted networks for downstream clustering, reducing false positives that plague raw Pearson thresholds.

## When to use

Use when integrating paired transcriptomic–metabolomic datasets (e.g., tomato samples from Jeon 2020) to find mass features correlated with biosynthetic genes without a prior 'bait', before clustering mass feature–transcript networks.

## When NOT to use

- When the goal is sequence/structure-based gene cluster identification rather than cross-omics correlation — use tools like plantiSMASH, PhytoClust, or PlantClusterFinder instead.
- When prior knowledge of a biosynthetic 'bait' enzyme or compound already anchors the pathway and a simple co-expression correlation would suffice.

## Inputs

- paired transcriptomic–metabolomic dataset (e.g., tomato, Jeon and colleagues 2020) with sample-matched gene expression and mass feature abundance tables
- MAD-rescaled expression and metabolite abundance matrices (after MEANtools MAD rescaling)

## Outputs

- per-network edge-weight files for DR = 5, 10, 25, and 50 exponential-decay networks
- per-network FC membership table listing transcripts and mass features assigned to each overlapping ClusterONE functional cluster (stored in SQLite)

## How to apply

Load the paired transcriptomic–metabolomic matrix and first rescale the input data using Median Absolute Deviation (MAD) as implemented in MEANtools. Compute Pearson correlation values between every transcript and mass feature across samples, then convert the pairwise correlation values to mutual ranks (MR) following the MEANtools MR-based correlation approach. Subject the continuous MR values to an exponential decay function to obtain discrete edge weights, generating networks with decay parameter DR = 5, 10, 25, and 50, and save each network's edge-weight file. The rationale is that integrating such datasets solely through Pearson-based correlations yields many false positives and optimal thresholds are hard to set; MR plus exponential decay yields robust discrete weights suitable for ClusterONE clustering. Feed the nodes and edge weights of each network into ClusterONE to produce overlapping feature clusters (FCs), storing results (e.g., in SQLite), and exclude gene–metabolite pairs with edge scores below 0.01 in the Ni→j networks before reaction-rule-based prediction steps.

## Related tools

- **MEANtools** (Implements MAD rescaling, MR-based correlation, exponential-decay edge weighting, and network generation from transcript–mass feature Pearson correlations) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters nodes and exponential-decay edge weights of the MR networks to produce multiple overlapping functional clusters (FCs))

## Examples

```
python meantools_mr_network.py --input jeon2020_paired.csv --mad-rescale --dr 5,10,25,50 --output edge_weights/
```

## Evaluation signals

- FC sizes prove stable across the four DR treatment combinations (DR = 5, 10, 25, 50) as reported for the Jeon and colleagues (2020) dataset (S7 Fig); verify all four DR networks yield FCs of comparable size.
- Known biosynthetic genes (e.g., PAL, 4CL, THT of the HCAA pathway; GAME genes of the alpha-tomatine pathway; HQT of the chlorogenic acid pathway) are recovered within resulting FCs.
- Gene–metabolite pairs with edge scores below 0.01 are correctly excluded in downstream Ni→j networks.
- The MR-to-edge-weight transformation produces discrete weights from continuous MR values for every transcript–mass feature pair (no missing edge weights).

## Limitations

- The FC size may sometimes be insufficient for forming a complete biosynthetic pathway.
- The current method to assign significance to each FC could be improved, as it was originally developed for co-expression datasets.
- Changing ClusterONE's inbuilt size parameter alters FC size but also changes the clustering pattern of mass features and transcripts.
- Pearson-based correlation integration alone yields many false-positive associations, which is why MR plus exponential decay is used rather than raw correlation thresholds.

## Evidence

- [discussion] MR correlation and clustering of nodes/weights: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers"
- [discussion] ClusterONE produces overlapping FCs: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] Pearson-only integration gives false positives: "Integrating such datasets solely through Pearson-based correlations often results in many false-positive associations. Additionally, determining an optimal threshold for eliminating weak correlations"
- [discussion] MAD rescaling is part of the MEANtools workflow: "MEANtools employs the Median Absolute Deviation (MAD) for data rescaling."
- [discussion] FC stability validation across DR treatments: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] Edge-score cutoff applied to gene–metabolite pairs: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
- [readme] README confirms MEANtools documentation and license: "MEANtools is licensed under the [MIT License](LICENSE.md)"
