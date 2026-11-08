---
name: spatial-cluster-differential-metabolite-analysis
description: Use when you have an annotated m/z intensity matrix paired with spatial
  cluster or region labels (e.g. tissue sub-structures) and need to identify m/z peaks
  that vary significantly between regions, producing a ranked table of differentially
  expressed metabolites.
when_to_use_negative:
- Input is already a pseudo-bulked or aggregated feature table rather than per-pixel
  m/z intensities with cluster labels
- The goal is metabolite annotation of raw m/z values rather than differential testing
  across existing clusters
- Data are single-cell resolution SM exported from METASPACE where MALDIpy-style workflows
  may be more appropriate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SpaMTP
  role: Provides the Seurat-class object wrapper over Cardinal data and implements
    the pseudo-bulked differential metabolite expression testing across spatial regions
    or clusters
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: edgeR
  role: Statistical engine for differential testing on pseudo-bulked m/z intensity
    values across groups
- name: Cardinal
  role: Reads the imzML MALDI imaging data into the m/z intensity matrix that SpaMTP
    consumes
- name: Seurat
  role: Foundational R package whose object class SpaMTP builds on for downstream
    statistical analyses including differential expression
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:ea4b28901daca591da0a5a01b7bb68c01019c068be5b5551259887f08e147d42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  - edgeR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-cluster-differential-metabolite-analysis

## Summary

SpaMTP-style differential metabolite peak expression testing that pseudo-bulks m/z intensities per spatial cluster or region and applies edgeR differential testing to identify and rank differentially expressed m/z peaks with fold-change and FDR statistics.

## When to use

Use when you have an annotated m/z intensity matrix paired with spatial cluster or region labels (e.g. tissue sub-structures) and need to identify m/z peaks that vary significantly between regions, producing a ranked table of differentially expressed metabolites.

## When NOT to use

- Input is already a pseudo-bulked or aggregated feature table rather than per-pixel m/z intensities with cluster labels
- The goal is metabolite annotation of raw m/z values rather than differential testing across existing clusters
- Data are single-cell resolution SM exported from METASPACE where MALDIpy-style workflows may be more appropriate

## Inputs

- Annotated m/z intensity matrix (e.g. from Cardinal readImzML of an imzML/ibd MALDI dataset)
- Spatial cluster or region labels per pixel (e.g. urothelium, muscle, adventitial layer)
- Comparison group specification (which clusters/regions to contrast)

## Outputs

- Ranked table of differentially expressed m/z peaks with fold-change and FDR statistics
- Annotation columns (metabolite names, e.g. PC(34:1), SM(34:1)) joined to the DEM table

## How to apply

Load the annotated m/z intensity matrix (from Cardinal via SpaMTP's Seurat-class object) together with spatial cluster labels such as urothelium, muscle, and adventitial layer from a mouse urinary bladder MALDI dataset. For each comparison group, randomly divide pixels into n pools with a fixed seed and sum relative intensity values for each m/z to create pseudo-bulked values — this aggregation stabilises per-region estimates before statistical testing. Apply edgeR differential testing on the pseudo-bulked intensities across the specified groups, computing per-m/z fold-change and false discovery rate. Retain significantly differentially expressed peaks and rank them by significance; the expected output for the bladder case is ~70 significant m/z peaks including PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle, matching the reported Table S5.

## Related tools

- **SpaMTP** (Provides the Seurat-class object wrapper over Cardinal data and implements the pseudo-bulked differential metabolite expression testing across spatial regions or clusters) — github.com/GenomicsMachineLearning/SpaMTP
- **edgeR** (Statistical engine for differential testing on pseudo-bulked m/z intensity values across groups)
- **Cardinal** (Reads the imzML MALDI imaging data into the m/z intensity matrix that SpaMTP consumes)
- **Seurat** (Foundational R package whose object class SpaMTP builds on for downstream statistical analyses including differential expression)

## Examples

```
SpaMTP::CalculateAnnotationStatistics(dem_table)
```

## Evaluation signals

- The DEM table should contain approximately 70 significantly differentially expressed m/z peaks between the three tissue-specific clusters in the mouse urinary bladder dataset
- PC(34:1) should appear as up-regulated in urothelium and SM(34:1) as up-regulated in muscle, matching Table S5
- Each retained peak row must carry both a fold-change and a false discovery rate value
- Peaks are ranked by significance in the output table
- The fixed seed used for pseudo-bulking should yield reproducible pool sums on re-run

## Limitations

- Pseudo-bulking depends on the chosen number of pools (n) and the random seed; different seeds may yield slightly different results
- EdgeR-style testing on pseudo-bulked intensities assumes the aggregated values behave like count-like data
- The workflow was demonstrated on MALDI data with cluster labels available (e.g. urothelium/muscle/adventitia); it requires pre-existing cluster definitions

## Evidence

- [methods] pseudo-bulked differential testing mechanism: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP integration with Cardinal and Seurat: [evidence span withheld — anchor status: external_doc]
- [results] SpaMTP published results: [evidence span withheld — anchor status: not_found]
