---
name: edger-differential-abundance-testing
description: Use when you have an annotated MALDI/MSI m/z intensity matrix plus spatial
  cluster labels (e.g. tissue regions) and need a reproducible ranked list of m/z
  peaks that differ significantly between those regions.
when_to_use_negative:
- Input is already a single-cell/single-ROI feature table from LC-MS with biological
  replicates — standard edgeR replicate designs apply instead of pixel pseudo-bulking.
- The goal is metabolite annotation of m/z values rather than testing for differential
  abundance — use SpaMTP's annotation pipeline (AnnotateBigData, BuildMZAnnotationIndex)
  instead.
- Clusters have not yet been defined on the MSI data — run SpaMTP's integrative clustering
  / SelectROIs first, since differential testing requires region labels.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: SpaMTP
  role: Provides the pseudo-bulking and differential metabolite expression testing
    workflow over the annotated intensity matrix and cluster labels
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: edgeR
  role: Statistical engine used for differential testing on the pseudo-bulked m/z
    intensities; installed as a Bioconductor dependency of SpaMTP
- name: Seurat
  role: Object foundation and cell/region-centric analyses from which SpaMTP's cluster
    labels and downstream statistical framework inherit
- name: Cardinal
  role: Reads the MALDI imzML data (e.g. readImzML) producing the intensity matrix
    that feeds the differential testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:8d3a235d9068c04e70200dd9b0c9d3ce318f9cc00d4df65fe06575cf8b29988a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# edger-differential-abundance-testing

## Summary

SpaMTP-style differential peak expression testing: m/z intensities are pseudo-bulked per spatial region/cluster (pixels randomly pooled with a fixed seed and summed) and tested with edgeR to produce a ranked table of differentially expressed metabolite peaks with fold-change and FDR statistics.

## When to use

Use when you have an annotated MALDI/MSI m/z intensity matrix plus spatial cluster labels (e.g. tissue regions) and need a reproducible ranked list of m/z peaks that differ significantly between those regions.

## When NOT to use

- Input is already a single-cell/single-ROI feature table from LC-MS with biological replicates — standard edgeR replicate designs apply instead of pixel pseudo-bulking.
- The goal is metabolite annotation of m/z values rather than testing for differential abundance — use SpaMTP's annotation pipeline (AnnotateBigData, BuildMZAnnotationIndex) instead.
- Clusters have not yet been defined on the MSI data — run SpaMTP's integrative clustering / SelectROIs first, since differential testing requires region labels.

## Inputs

- Annotated m/z relative-intensity matrix (e.g. RDS/imzML-derived SpaMTP/Cardinal object)
- Per-pixel spatial cluster or region labels (e.g. urothelium, muscle, adventitial layer)
- Comparison-group specification (which clusters to contrast, number of pools n, random seed)

## Outputs

- Ranked table of differentially expressed m/z peaks with fold-change, FDR, and annotation columns (dem_results_table.csv)

## How to apply

Load the annotated m/z relative-intensity matrix (from Cardinal/SpaMTP objects, e.g. an RDS of MALDI data) together with tissue-cluster labels such as urothelium, muscle, and adventitial layer. For each pairwise comparison, randomly divide pixels into n pools using a fixed random seed and sum the relative intensities of each m/z within pools to create pseudo-bulked values, which stabilises pixel-level noise before testing. Run edgeR differential testing on the pseudo-bulked intensity values across the specified groups, then compute per-m/z statistics including fold-change and false discovery rate, retaining significantly differentially expressed peaks. Rank peaks by significance and write the DEM table (with metabolite annotation columns) to dem_results_table.csv. Validation target from the mouse urinary bladder MALDI dataset: ~70 significantly differentially expressed m/z peaks across the three tissue clusters, with PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle, as reported in Table S5.

## Related tools

- **SpaMTP** (Provides the pseudo-bulking and differential metabolite expression testing workflow over the annotated intensity matrix and cluster labels) — https://github.com/GenomicsMachineLearning/SpaMTP
- **edgeR** (Statistical engine used for differential testing on the pseudo-bulked m/z intensities; installed as a Bioconductor dependency of SpaMTP)
- **Seurat** (Object foundation and cell/region-centric analyses from which SpaMTP's cluster labels and downstream statistical framework inherit)
- **Cardinal** (Reads the MALDI imzML data (e.g. readImzML) producing the intensity matrix that feeds the differential testing)

## Examples

```
if (!require("SpaMTP")) devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- dem_results_table.csv contains per-m/z rows with fold-change, FDR, and annotation columns, ranked by significance.
- Reproduces ~70 significantly differentially expressed m/z peaks between the three tissue-specific clusters on the mouse urinary bladder MALDI dataset.
- Directionality check: PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle, matching Table S5.
- Fixed seed makes the pixel-to-pool assignment reproducible; re-running yields an identical DEM table.
- Every retained peak carries an annotation entry, confirming the input matrix was the annotated (not raw) intensity matrix.

## Limitations

- Pseudo-bulking depends on the random pixel-to-pool split; the fixed seed must be recorded or results will vary between runs.
- Downstream differential methodologies common in ST packages are incompatible with raw Cardinal objects — the data must pass through SpaMTP's Seurat-based object first.
- No explicit SpaMTP version number is stated for the workflow in the source material; record the installed version when reproducing.
- Paired targeted metabolomics comparison support in the vignette is a placeholder ('To come!'), so paired-design testing is not covered.

## Evidence

- [other] Run edgeR differential testing on the pseudo-bulked intensity values across the specified groups.: [evidence span withheld — anchor status: not_found]
- [other] For each comparison group, randomly divide pixels into n pools with a fixed seed and sum relative intensity values for each m/z (pseudo-bulking).: [evidence span withheld — anchor status: not_found]
- [readme] various downstream statistical analysis including differential metabolite expression and pathway analysis: [evidence span withheld — anchor status: external_doc]
