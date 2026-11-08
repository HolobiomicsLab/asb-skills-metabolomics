---
name: pseudo-bulk-pooling-of-spatial-metabolite-intensities
description: Use when you have an annotated m/z intensity matrix with tissue-cluster
  labels (e.g. urothelium, muscle, adventitial layer) and need to identify metabolite
  peaks that vary between spatial regions via differential expression testing.
when_to_use_negative:
- When the input is already bulk or pseudo-bulked metabolomics data rather than pixel-level
  MALDI imaging intensities requiring aggregation.
- When the goal is multi-modal SM-ST integration or pathway analysis rather than region-vs-region
  differential peak expression testing.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3520
tools:
- name: SpaMTP
  role: R package that aggregates pixel-level MALDI intensities into pseudo-bulked
    pools and runs differential expression testing across spatial clusters
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: edgeR
  role: Differential testing engine applied to the pseudo-bulked intensity values
    across spatial regions
- name: Cardinal
  role: Imports MALDI imzML data and provides the m/z intensity matrix that SpaMTP
    pseudo-bulks and tests
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:e3eb8dd7c1ed3254defc1c9cb549707f7b63b2c0bcfb1a03bfb4643d03b69618
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  - edgeR
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Pseudo-bulk pooling of spatial metabolite intensities

## Summary

Aggregate MALDI imaging m/z intensity values per spatial region or cluster by summing relative intensities into pseudo-bulked pools, then run edgeR differential testing to identify significantly differentially expressed metabolite peaks between tissue regions.

## When to use

Use when you have an annotated m/z intensity matrix with tissue-cluster labels (e.g. urothelium, muscle, adventitial layer) and need to identify metabolite peaks that vary between spatial regions via differential expression testing.

## When NOT to use

- When the input is already bulk or pseudo-bulked metabolomics data rather than pixel-level MALDI imaging intensities requiring aggregation.
- When the goal is multi-modal SM-ST integration or pathway analysis rather than region-vs-region differential peak expression testing.

## Inputs

- Annotated m/z intensity matrix from MALDI imaging data (e.g. imzML-imported via Cardinal into a SpaMTP/Seurat object)
- Per-pixel spatial cluster or region-of-interest labels (e.g. urothelium, muscle, adventitial layer)

## Outputs

- Ranked table of differentially expressed m/z peaks with fold-change and false discovery rate statistics (e.g. dem_results_table.csv)

## How to apply

Load the annotated m/z intensity matrix and spatial cluster labels for each pixel from a SpaMTP (Seurat-style) spatial metabolomics object. For each comparison group, randomly divide pixels into n pools with a fixed seed and sum the relative intensity values for each m/z, producing pseudo-bulked pools analogous to bulk metabolomics measurements. Run edgeR differential testing on these pseudo-bulked intensities across the specified groups to compute per-m/z fold-change and false discovery rate statistics. Retain significantly differentially expressed peaks and rank them by significance; SpaMTP demonstrated this on a mouse urinary bladder MALDI dataset, recovering ~70 differentially expressed m/z peaks between three tissue-specific clusters, including PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle.

## Related tools

- **SpaMTP** (R package that aggregates pixel-level MALDI intensities into pseudo-bulked pools and runs differential expression testing across spatial clusters) — github.com/GenomicsMachineLearning/SpaMTP
- **edgeR** (Differential testing engine applied to the pseudo-bulked intensity values across spatial regions)
- **Cardinal** (Imports MALDI imzML data and provides the m/z intensity matrix that SpaMTP pseudo-bulks and tests)

## Examples

```
Rscript -e 'SpaMTP::FindAllDEMs(spa_obj, group.by="tissue_cluster", n_pools=3, seed=42)'
```

## Evaluation signals

- The output DEM table should contain per-m/z rows with fold-change and false discovery rate columns.
- On the mouse urinary bladder MALDI dataset, the table should reproduce ~70 significantly differentially expressed m/z peaks between the three tissue-specific clusters.
- Known biology should be recovered: PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle, as reported in Table S5.
- Pseudo-bulked pool counts must be reproducible: a fixed seed governs the random division of pixels into n pools per group.
- Significantly differentially expressed peaks should be ranked by significance in the output table.

## Limitations

- Pseudo-bulking discards pixel-level spatial heterogeneity within each region, so results reflect region-level bulk differences only.
- Testing is performed on summed relative intensities, so absolute abundance differences and within-pool variance structure may be lost.
- Results depend on the quality of upstream spatial clustering; misassigned pixels propagate into the pseudo-bulked pools.

## Evidence

- [methods] Fixed-seed pooling of pixels per group: [evidence span withheld — anchor status: not_found]
- [methods] ~70 DEMs reproduce Table S5: [evidence span withheld — anchor status: not_found]
- [results] Published as differential expression of m/z peaks: [evidence span withheld — anchor status: not_found]
