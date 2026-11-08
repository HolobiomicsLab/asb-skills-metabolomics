---
name: specialized-metabolism-pathway-curation
description: Use when a MEANtools/ClusterONE analysis of paired transcriptomic-metabolomic
  data (e.g., Jeon 2020 tomato) yields FCs and you need to assign known pathway genes
  to clusters and check cluster stability.
when_to_use_negative:
- When the goal is de novo reaction-step reconstruction rather than gene-to-cluster
  curation — MEANtools prediction was hindered for PAL/THT FCs by stereoisomers and
  two-substrate reactions (e.g., p-coumaroyltyramine needs tyrosine and p-coumaroyl-CoA)
  that RetroRules single-substrate rules cannot represent
- When MS/MS-based structural annotation is required — MEANtools currently does not
  support loading MS/MS data or spectral library/networking-based annotation
- When input is a single co-expression table without paired metabolomic mass features
  — FC significance assessment was originally developed for co-expression datasets
  and may not transfer
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3308
tools:
- name: MEANtools
  role: 'Integrative pipeline: MAD rescaling, MR-based correlation, exponential-decay
    edge weights, and FC generation; provides merge_clusters.py for merging overlapping
    FCs'
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters transcript and mass-feature nodes with exponential-decay edge weights
    to produce multiple overlapping FCs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:64bbd6b1492e6d80fe343fe6e0e33fbed72001397f46c1528dfeb5e71a7febc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# specialized-metabolism-pathway-curation

## Summary

Curation skill for mapping known biosynthetic genes of tomato specialized metabolism pathways (HCAA: PAL/4CL/THT; alpha-tomatine: GAME1/4/6/7/9/11/12/17/18, SQS, TTS1/2, SSR2; chlorogenic acid: HQT) onto overlapping MEANtools feature clusters (FCs) derived from a paired transcriptomic-metabolomic dataset, and tabulating pathway/gene/FC memberships for validation.

## When to use

Use when a MEANtools/ClusterONE analysis of paired transcriptomic-metabolomic data (e.g., Jeon 2020 tomato) yields FCs and you need to assign known pathway genes to clusters and check cluster stability.

## When NOT to use

- When the goal is de novo reaction-step reconstruction rather than gene-to-cluster curation — MEANtools prediction was hindered for PAL/THT FCs by stereoisomers and two-substrate reactions (e.g., p-coumaroyltyramine needs tyrosine and p-coumaroyl-CoA) that RetroRules single-substrate rules cannot represent
- When MS/MS-based structural annotation is required — MEANtools currently does not support loading MS/MS data or spectral library/networking-based annotation
- When input is a single co-expression table without paired metabolomic mass features — FC significance assessment was originally developed for co-expression datasets and may not transfer

## Inputs

- Jeon 2020 paired tomato transcriptomic-metabolomic dataset (deposited source files)
- MEANtools-rescaled expression/mass-feature matrices (MAD-normalized)
- Curated gene lists for known pathways: PAL, 4CL, THT (HCAA); GAME1/4/6/7/9/11/12/17/18, SQS, TTS1, TTS2, SSR2 (alpha-tomatine); HQT (chlorogenic acid)

## Outputs

- biosynthetic_gene_fc_membership.csv with pathway/gene/FC columns
- Overlapping feature clusters (FCs) from ClusterONE, merged with merge_clusters.py where they share common mass features

## How to apply

Load the Jeon 2020 paired tomato transcriptomic-metabolomic dataset from the deposited source files, rescale it with Median Absolute Deviation (MAD) normalization, convert transcript–mass feature Pearson correlations to mutual ranks and exponential-decay edge weights, and cluster nodes and edges with ClusterONE to produce overlapping FCs, merging FCs that share common mass features with merge_clusters.py. Then map known biosynthetic genes — HCAA (PAL, 4CL, THT), alpha-tomatine (GAME1/4/6/7/9/11/12/17/18, SQS, TTS1, TTS2, SSR2), and chlorogenic acid (HQT) — onto the FCs, noting that overlapping clusters may place a single gene in more than one FC. Write the membership table to biosynthetic_gene_fc_membership.csv with pathway, gene, and FC columns. Judge correctness by verifying the file exists with all three fields populated, that at least one representative gene from each of the three pathways appears, and that FC assignments are stable across treatment combinations as reported in S7 Fig.

## Related tools

- **MEANtools** (Integrative pipeline: MAD rescaling, MR-based correlation, exponential-decay edge weights, and FC generation; provides merge_clusters.py for merging overlapping FCs) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters transcript and mass-feature nodes with exponential-decay edge weights to produce multiple overlapping FCs)

## Examples

```
gene_fc_df.to_csv('biosynthetic_gene_fc_membership.csv', columns=['pathway','gene','FC'], index=False); # after mapping PAL/4CL/THT, GAME/SQS/TTS/SSR2, HQT onto overlapping ClusterONE FCs
```

## Evaluation signals

- biosynthetic_gene_fc_membership.csv exists and all pathway/gene/FC fields are populated (no empty cells)
- Representative genes from each of the three pathways appear: at least one of PAL/4CL/THT, one of GAME1–GAME18/SQS/TTS1/TTS2/SSR2, and HQT
- FC sizes and assignments remain stable across treatment combinations, matching the S7 Fig report for the Jeon 2020 dataset
- Genes appearing in multiple FCs are correctly retained as duplicate memberships, since ClusterONE FCs overlap by design
- Edge weights feeding clustering derive from MR values after exponential decay, with gene–metabolite pairs below an edge score of 0.01 excluded

## Limitations

- FC size may sometimes be insufficient for forming a complete biosynthetic pathway
- The current FC significance method was originally developed for co-expression datasets and could be improved
- Changing ClusterONE's inbuilt size parameter also changes the clustering pattern of mass features and transcripts, so FC-size tuning is coupled across node types
- Mass spectrometric data do not capture stereoisomers, and RetroRules rules are designed for single-substrate reactions only, hindering pathway reconstruction for PAL and THT FCs

## Evidence

- [discussion] ClusterONE overlaps: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] merge_clusters.py: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] S7 Fig stability: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [discussion] MAD rescaling: "MEANtools employs the Median Absolute Deviation (MAD) for data rescaling."
- [abstract] Hypothesis generation on specialized metabolism: "also identified other candidate pathways involved in specialized metabolism, which demonstrates its potential for hypothesis generation"
- [readme] README documentation pointer: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
