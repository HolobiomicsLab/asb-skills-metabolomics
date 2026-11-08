---
name: biosynthetic-gene-to-cluster-mapping
description: Use when MEANtools/ClusterONE has produced overlapping feature clusters
  from a paired transcriptome-metabolome dataset and you need to annotate which FCs
  contain biosynthetic genes from known pathways to validate clustering or generate
  hypotheses.
when_to_use_negative:
- When the goal is de novo pathway reconstruction via reaction rules (RetroRules/LOTUS)
  rather than validating clustering against known gene sets — MEANtools' prediction
  step handles that separately
- When no prior biosynthetic gene knowledge exists for the pathway of interest, since
  this skill relies on curated gene lists as mapping targets
- When the dataset has not been clustered into FCs yet (MAD rescaling, MR correlation,
  and ClusterONE must run first)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MEANtools
  role: Generates the overlapping feature clusters (after MAD rescaling, MR-based
    correlation with exponential-decay weights) and provides merge_clusters.py for
    merging FCs sharing mass features, onto which biosynthetic genes are mapped
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters nodes (transcripts and mass features) and edge weights into multiple
    overlapping feature clusters that are the mapping targets for biosynthetic genes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:553a7bb2d589bb396d10fe51e525ea1d4114e6a90bb4740aa0f8647c876a0b87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# biosynthetic-gene-to-cluster-mapping

## Summary

Maps known biosynthetic genes from specialized metabolism pathways (e.g., HCAA, alpha-tomatine, chlorogenic acid) onto MEANtools feature clusters (FCs) to verify that unsupervised clustering recovers established tomato metabolic pathways in the Jeon 2020 paired transcriptomic-metabolomic dataset.

## When to use

Use when MEANtools/ClusterONE has produced overlapping feature clusters from a paired transcriptome-metabolome dataset and you need to annotate which FCs contain biosynthetic genes from known pathways to validate clustering or generate hypotheses.

## When NOT to use

- When the goal is de novo pathway reconstruction via reaction rules (RetroRules/LOTUS) rather than validating clustering against known gene sets — MEANtools' prediction step handles that separately
- When no prior biosynthetic gene knowledge exists for the pathway of interest, since this skill relies on curated gene lists as mapping targets
- When the dataset has not been clustered into FCs yet (MAD rescaling, MR correlation, and ClusterONE must run first)

## Inputs

- MEANtools feature clusters (FCs) produced by ClusterONE on MR-based exponential-decay edge weights from a paired transcriptomic-metabolomic dataset (e.g., Jeon 2020 tomato)
- Curated lists of known biosynthetic genes per pathway (e.g., PAL, 4CL, THT for HCAA; GAME1/4/6/7/9/11/12/17/18, SQS, TTS1/2, SSR2 for alpha-tomatine; HQT for chlorogenic acid)
- Merged FC definitions from merge_clusters.py (overlapping clusters sharing mass features)

## Outputs

- biosynthetic_gene_fc_membership.csv with columns pathway, gene, FC membership
- Per-pathway recovery summary indicating which known biosynthetic genes appear in which FCs

## How to apply

First run the MEANtools upstream pipeline: load the paired dataset, rescale with Median Absolute Deviation (MAD), convert transcript–mass feature Pearson correlations to mutual ranks (MRs) and exponential-decay edge weights, then cluster nodes and edges with ClusterONE to yield multiple overlapping feature clusters, merging FCs that share common mass features with merge_clusters.py. Next, compile the known biosynthetic gene lists for each pathway of interest — for tomato: HCAA (PAL, 4CL, THT), alpha-tomatine (GAME1, GAME4, GAME6, GAME7, GAME9, GAME11, GAME12, GAME17, GAME18, SQS, TTS1, TTS2, SSR2), and chlorogenic acid (HQT). Map each gene onto the resulting FCs, noting that overlapping clusters may assign a single gene to more than one FC. Write a membership table with columns for pathway, gene, and FC assignment (e.g., biosynthetic_gene_fc_membership.csv) and check stability across treatment combinations, as reported in S7 Fig.

## Related tools

- **MEANtools** (Generates the overlapping feature clusters (after MAD rescaling, MR-based correlation with exponential-decay weights) and provides merge_clusters.py for merging FCs sharing mass features, onto which biosynthetic genes are mapped) — github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters nodes (transcripts and mass features) and edge weights into multiple overlapping feature clusters that are the mapping targets for biosynthetic genes)

## Examples

```
python merge_clusters.py --clusters clusterone_output/ --output merged_fcs/ && python map_genes_to_fcs.py --genes biosynthetic_genes.csv --fcs merged_fcs/ --out biosynthetic_gene_fc_membership.csv
```

## Evaluation signals

- biosynthetic_gene_fc_membership.csv exists with pathway, gene, and FC fields populated
- Representative genes from each of the three pathways (e.g., PAL/THT for HCAA, GAME1/GAME17 for alpha-tomatine, HQT for chlorogenic acid) appear in the table
- FC assignments are stable across treatment combinations, consistent with S7 Fig findings in the Jeon 2020 dataset
- Genes appearing in multiple FCs are consistently recorded, reflecting the overlapping-cluster structure from ClusterONE

## Limitations

- Overlapping clusters may assign a gene to more than one FC, so membership is not exclusive
- FC size may sometimes be insufficient for forming a complete biosynthetic pathway
- The current method to assign significance to each FC was originally developed for co-expression datasets and could be improved
- Changing ClusterONE parameters to alter FC size also changes the clustering pattern of mass features and transcripts, affecting mapping results

## Evidence

- [discussion] Cluster nodes and edge weights with ClusterONE to produce multiple overlapping feature clusters (FCs), then merge FCs sharing common mass features with merge_clusters.py.: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] MEANtools provides a script to merge FCs sharing mass features.: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] FC size stable across treatment combinations in the Jeon 2020 dataset.: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
- [abstract] MEANtools is a systematic unsupervised integrative omics workflow.: "Here, we present MEANtools, a systematic and unsupervised computational integrative omics workflow"
- [readme] Repository README confirms MEANtools documentation and maintainer.: "MEANtools is licensed under the [MIT License](LICENSE.md)"
