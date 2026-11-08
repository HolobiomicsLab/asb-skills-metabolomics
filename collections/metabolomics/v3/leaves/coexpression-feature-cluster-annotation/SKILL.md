---
name: coexpression-feature-cluster-annotation
description: Use when a MEANtools run on paired transcriptomic-metabolomic data has
  yielded FCs and you need to check whether genes from known pathways (e.g., GAME,
  PAL, 4CL, THT, HQT) land in them, or to benchmark clustering against known tomato
  specialized metabolism.
when_to_use_negative:
- When the goal is de novo discovery of unknown biosynthetic pathways rather than
  validating FC content against known pathway genes — for that use the MEANtools prediction
  step with RetroRules/LOTUS instead.
- When the input lacks paired transcriptomic-metabolomic samples from the same conditions,
  since MR-based correlation between transcripts and mass features cannot be computed.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MEANtools
  role: Pipeline that performs MAD rescaling, MR-based correlation with exponential-decay
    edge weights, and provides merge_clusters.py for merging FCs sharing mass features;
    the FCs being annotated are its output.
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters nodes (transcripts and mass features) and edge weights to produce
    the multiple overlapping feature clusters (FCs) onto which biosynthetic genes
    are mapped.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:23baa5ee4627000d2ada5a63e50d1e7794bfb9034107faff056f252454e31372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - MEANtools
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# coexpression-feature-cluster-annotation

## Summary

Maps known biosynthetic genes (e.g., tomato HCAA, alpha-tomatine, and chlorogenic acid pathway genes) onto ClusterONE-derived overlapping feature clusters (FCs) produced by MEANtools, producing a gene-to-FC membership table that shows whether established specialized metabolism pathways co-cluster with correlated mass features.

## When to use

Use when a MEANtools run on paired transcriptomic-metabolomic data has yielded FCs and you need to check whether genes from known pathways (e.g., GAME, PAL, 4CL, THT, HQT) land in them, or to benchmark clustering against known tomato specialized metabolism.

## When NOT to use

- When the goal is de novo discovery of unknown biosynthetic pathways rather than validating FC content against known pathway genes — for that use the MEANtools prediction step with RetroRules/LOTUS instead.
- When the input lacks paired transcriptomic-metabolomic samples from the same conditions, since MR-based correlation between transcripts and mass features cannot be computed.

## Inputs

- Paired transcriptomic-metabolomic dataset (e.g., Jeon 2020 tomato deposited source files)
- MEANtools FC assignments from ClusterONE clustering (optionally merged with merge_clusters.py)
- Curated list of known biosynthetic genes per pathway (e.g., PAL, 4CL, THT for HCAA; GAME1/4/6/7/9/11/12/17/18, SQS, TTS1/2, SSR2 for alpha-tomatine; HQT for chlorogenic acid)

## Outputs

- biosynthetic_gene_fc_membership.csv with columns for pathway, gene, and FC membership
- Per-FC annotation of which known specialized metabolism pathways are represented

## How to apply

Load the paired transcriptomic-metabolomic dataset (e.g., Jeon 2020 tomato), rescale with Median Absolute Deviation (MAD) normalization, convert Pearson transcript-mass feature correlations to mutual ranks (MR), transform MRs with an exponential decay function to edge weights, and cluster nodes and weights with ClusterONE to produce overlapping FCs; merge FCs sharing common mass features with merge_clusters.py. Then compile the known biosynthetic genes of each pathway of interest — HCAA (PAL, 4CL, THT), alpha-tomatine (GAME1, GAME4, GAME6, GAME7, GAME9, GAME11, GAME12, GAME17, GAME18, SQS, TTS1, TTS2, SSR2), and chlorogenic acid (HQT) — and map them onto the resulting FCs, noting that overlapping clusters may assign a gene to more than one FC. Write a table with columns for pathway, gene, and FC membership. Gene-metabolite pairs with edge score below 0.01 are excluded in the Ni→j networks, so verify gene-FC calls against the retained edges.

## Related tools

- **MEANtools** (Pipeline that performs MAD rescaling, MR-based correlation with exponential-decay edge weights, and provides merge_clusters.py for merging FCs sharing mass features; the FCs being annotated are its output.) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters nodes (transcripts and mass features) and edge weights to produce the multiple overlapping feature clusters (FCs) onto which biosynthetic genes are mapped.)

## Examples

```
python merge_clusters.py --input clusters.tsv --output merged_fcs.tsv  # then map pathway genes onto merged FCs and write biosynthetic_gene_fc_membership.csv
```

## Evaluation signals

- biosynthetic_gene_fc_membership.csv exists with pathway/gene/FC fields populated.
- Representative genes from each of the three pathways (HCAA, alpha-tomatine, chlorogenic acid) appear in the table.
- FC assignments are stable across treatment combinations, as reported in S7 Fig of the Jeon 2020 analysis.
- Genes appearing in multiple FCs are retained and flagged, since overlapping clusters may assign a gene to more than one FC.
- Only gene-metabolite pairs with edge score ≥ 0.01 contribute to the FCs being annotated.

## Limitations

- Changing ClusterONE parameters to alter FC size also changes the clustering pattern of mass features and transcripts, which can move genes between FCs.
- The FC size may sometimes be insufficient for forming a complete biosynthetic pathway, so absence of some pathway genes in an FC does not necessarily invalidate the cluster.
- Overlapping clusters may assign a gene to more than one FC, so one-to-one gene-FC mappings are not guaranteed.

## Evidence

- [results] In the Jeon 2020 dataset, MEANtools found multiple FCs containing biosynthetic genes from the HCAA pathway (PAL, 4CL, THT), the alpha-tomatine pathway (GAME1, GAME4, GAME6, GAME7, GAME11, GAME12, GAME17, GAME18, plus GAME9, SQS, TTS1, TTS2, SSR2), and the chlorogenic acid pathway (HQT).: [evidence span withheld — anchor status: not_found]
- [discussion] Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs.: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features.: "MEANtools provides a script (merge_clusters.py) to merge multiple FCs that share common mass features."
- [discussion] MEANtools employs the Median Absolute Deviation (MAD) for data rescaling.: "MEANtools employs the Median Absolute Deviation (MAD) for data rescaling."
- [discussion] The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig).: "The FCs size proved stable across the treatment combinations in Jeon and colleagues (2020) dataset [30] (S7 Fig)."
