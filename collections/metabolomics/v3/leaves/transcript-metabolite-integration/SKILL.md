---
name: transcript-metabolite-integration
description: Use when a paired transcriptomic-metabolomic dataset must be integrated
  without a known 'bait' metabolite or enzyme to surface feature clusters containing
  biosynthetic genes.
when_to_use_negative:
- Prior knowledge of a bait compound or enzyme is already available and a targeted
  bait-based co-expression approach suffices
- The input is not a paired transcriptomic-metabolomic dataset with matched samples
  (MR correlation and FC construction require both data types)
- The goal is enzyme-level pathway reconstruction requiring multi-substrate or stereospecific
  reactions, which RetroRules single-substrate rules and MS data cannot capture
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: MEANtools
  role: 'Main pipeline: MAD rescaling, MR-based correlation with exponential-decay
    edge weights, adduct/LOTUS structure annotation, merge_clusters.py, and FC significance'
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters node types (transcripts, mass features) and edge weights into multiple
    overlapping feature clusters (FCs)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:8e051fd2c6b207af16afc58a3c3175ae9d4499c2e503ecf621255edbde0199b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# transcript-metabolite-integration

## Summary

Unsupervised integration of paired transcriptomic and metabolomic data using MEANtools: MAD-rescaled inputs, mutual rank (MR)-based correlations converted to exponential-decay edge weights, and ClusterONE clustering into overlapping feature clusters (FCs) that can be mapped to known biosynthetic genes (e.g., HCAA, alpha-tomatine, chlorogenic acid pathways in tomato).

## When to use

Use when a paired transcriptomic-metabolomic dataset must be integrated without a known 'bait' metabolite or enzyme to surface feature clusters containing biosynthetic genes.

## When NOT to use

- Prior knowledge of a bait compound or enzyme is already available and a targeted bait-based co-expression approach suffices
- The input is not a paired transcriptomic-metabolomic dataset with matched samples (MR correlation and FC construction require both data types)
- The goal is enzyme-level pathway reconstruction requiring multi-substrate or stereospecific reactions, which RetroRules single-substrate rules and MS data cannot capture

## Inputs

- Paired transcriptomic-metabolomic dataset (e.g., Jeon 2020 tomato paired transcriptomic-metabolomic dataset) as deposited source files
- Gene expression matrix (transcripts x samples)
- Mass feature abundance matrix (metabolites x samples)
- LOTUS metabolite database in SQLite format (e.g., downloaded 10/10/2023) for structure annotation
- RetroRules reaction-rule datasets (loose/medium/strict) as CSV (e.g., Combined_small_middle_big_datasets.csv from Zenodo 15697913)

## Outputs

- Overlapping feature clusters (FCs) of transcripts and mass features with edge weights
- Merged FCs produced by merge_clusters.py
- Weighted transcript-metabolite network (Ni->j networks) with edge scores above 0.01
- Biosynthetic gene-to-FC membership table (biosynthetic_gene_fc_membership.csv with pathway/gene/FC columns)

## How to apply

Load the paired transcriptomic-metabolomic dataset (e.g., the Jeon 2020 tomato dataset) and rescale both data types using Median Absolute Deviation (MAD) as implemented in MEANtools. Compute pairwise transcript-mass feature correlations, convert Pearson values to mutual ranks (MRs), then transform MRs with an exponential decay function into edge weights, excluding gene-metabolite pairs below the 0.01 edge score cutoff. Cluster both node types and edge weights with ClusterONE to generate multiple overlapping feature clusters (FCs), and merge FCs sharing common mass features with MEANtools' merge_clusters.py script. Map known biosynthetic genes (e.g., PAL/4CL/THT for HCAA; GAME1/4/6/7/9/11/12/17/18, SQS, TTS1/2, SSR2 for alpha-tomatine; HQT for chlorogenic acid) onto the FCs, noting that overlapping clusters may assign a gene to more than one FC. Optionally, annotate mass features to metabolite structures by adduct identification at a default 20 ppm tolerance and querying the LOTUS database.

## Related tools

- **MEANtools** (Main pipeline: MAD rescaling, MR-based correlation with exponential-decay edge weights, adduct/LOTUS structure annotation, merge_clusters.py, and FC significance) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters node types (transcripts, mass features) and edge weights into multiple overlapping feature clusters (FCs))

## Evaluation signals

- biosynthetic_gene_fc_membership.csv exists with pathway, gene, and FC fields populated
- Representative genes from each of the three known pathways appear in the membership table: PAL/4CL/THT (HCAA), GAME/SQS/TTS1/TTS2/SSR2 (alpha-tomatine), HQT (chlorogenic acid)
- FC assignments are stable across treatment combinations, as reported in S7 Fig of the Jeon 2020 dataset analysis
- Gene-metabolite pairs with edge score below 0.01 are absent from the Ni->j networks
- FCs that share common mass features have been merged via merge_clusters.py rather than kept as redundant separate clusters

## Limitations

- FC size may be insufficient to form a complete biosynthetic pathway
- The current FC significance method was originally developed for co-expression datasets and could be improved
- Changing ClusterONE's inbuilt size parameters alters the clustering pattern of both mass features and transcripts
- Mass spectrometric data does not capture stereoisomers, hindering reconstruction of steps like phenylalanine to p-coumaroyl-CoA
- RetroRules-based rules handle single-substrate reactions only, blocking steps like p-coumaroyltyramine formation from two substrates

## Evidence

- [abstract] Here, we present MEANtools, a systematic and unsupervised computational integrative omics workflow: "Here, we present MEANtools, a systematic and unsupervised computational integrative omics workflow"
- [intro] MEANtools implements a mutual rank (MR)-based correlation approach to capture mass features correlated with biosynthetic genes: "MEANtools implements a mutual rank (MR)-based [14] correlation approach to capture mass features that are highly correlated with biosynthetic genes."
- [discussion] MEANtools employs the Median Absolute Deviation (MAD) for data rescaling: "MEANtools employs the Median Absolute Deviation (MAD) for data rescaling."
- [discussion] Both node types and edge weights are further subjected to clustering using ClusterONE, which results in multiple overlapping FCs: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] Gene-metabolite pairs that show lower edge score than 0.01 are excluded in the Ni->j networks: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
