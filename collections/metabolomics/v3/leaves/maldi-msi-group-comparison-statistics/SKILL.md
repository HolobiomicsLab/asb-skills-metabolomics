---
name: maldi-msi-group-comparison-statistics
description: Use when you have segmented, ROI-annotated MSI pixel-intensity data from
  two sample groups and need per-lipid fold-change/p-value statistics per tissue region
  with volcano plots.
when_to_use_negative:
- You only need ROI segmentation or cell-population segmentation from raw MSI data
  (use msi_segmentation_flow or if_segmentation_flow instead)
- You need ML-based classification of molecular signatures rather than two-group statistical
  testing (use molecular_signatures_flow)
- Input is raw imzML data with no peak list or segmentation produced yet (pre-processing
  via msi_preprocessing_flow must run first)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: Provides region_group_analysis_flow, the Snakemake workflow that performs
    pixel-wise statistical testing of mean lipid intensities between two groups within
    each ROI
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Command-line workflow engine used to execute region_group_analysis_flow with
    a configfile and parallel cores
- name: msi_segmentation_flow
  role: Upstream msiFlow workflow that produces the segmented, ROI-annotated pixel-intensity
    table consumed by the group comparison
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Container image packaging msiFlow workflows so they run platform-independently
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:6e363f68ebfce8b1da306065483947bbcc7311c06672edff0fd6a7e1ac342b84
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  repo_url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Docker
  - Snakemake
  - msiFlow
  - msi_segmentation_flow
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# maldi-msi-group-comparison-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical comparison of MALDI MSI pixel intensities between two biological groups (e.g. UPEC-infected vs control) within segmented tissue ROIs, using msiFlow's region_group_analysis_flow to identify regulated lipid classes and visualise them as volcano plots.

## When to use

Use when you have segmented, ROI-annotated MSI pixel-intensity data from two sample groups and need per-lipid fold-change/p-value statistics per tissue region with volcano plots.

## When NOT to use

- You only need ROI segmentation or cell-population segmentation from raw MSI data (use msi_segmentation_flow or if_segmentation_flow instead)
- You need ML-based classification of molecular signatures rather than two-group statistical testing (use molecular_signatures_flow)
- Input is raw imzML data with no peak list or segmentation produced yet (pre-processing via msi_preprocessing_flow must run first)

## Inputs

- Segmented, ROI-annotated MSI pixel-intensity table from msi_segmentation_flow (pandas-readable)
- Group metadata mapping samples to infected (n=4) and control (n=4) groups
- Pixel-to-ROI mapping (urothelium, lamina propria, muscle)
- YAML configuration file for the Snakemake workflow

## Outputs

- Per-lipid log2 fold-change and p-value table per ROI (urothelium, lamina propria, muscle)
- Ranked regulated-lipid table with effect direction and lipid-class annotation
- Volcano plot (log2 FC vs -log10 p-value) per ROI highlighting TG, DG, PC, PE and ST species
- Workflow output files in the region_group_analysis_flow output directory

## How to apply

Load the segmented, ROI-annotated MSI pixel-intensity table produced by msi_segmentation_flow with pandas, then attach group metadata (e.g. infected n=4 vs control n=4) and map pixels to urothelium, lamina propria and muscle ROIs. Run msiFlow's region_group_analysis_flow via its Snakemake CLI (`snakemake --snakefile region_group_analysis_flow/Snakefile --cores <cores> --configfile <config>`), which performs pixel-wise statistical testing of mean lipid intensities between the two groups within each ROI. Aggregate per-lipid log2 fold-changes and p-values into a regulated-lipid table for each ROI, ranking lipids by significance and effect direction. Generate volcano plots (log2 fold change vs -log10 p-value) per ROI with matplotlib, highlighting significantly regulated species (p < 0.05, |log2(FC)| > 1) and their lipid classes such as TG, DG, PC, PE and ST.

## Related tools

- **msiFlow** (Provides region_group_analysis_flow, the Snakemake workflow that performs pixel-wise statistical testing of mean lipid intensities between two groups within each ROI) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Command-line workflow engine used to execute region_group_analysis_flow with a configfile and parallel cores)
- **msi_segmentation_flow** (Upstream msiFlow workflow that produces the segmented, ROI-annotated pixel-intensity table consumed by the group comparison) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Container image packaging msiFlow workflows so they run platform-independently)

## Examples

```
snakemake --snakefile region_group_analysis_flow/Snakefile --cores 8 --configfile config.yaml
```

## Evaluation signals

- Output table and volcano plot reproduce the reported regulation directions: TG/DG/PC species up-regulated and PE/ST species down-regulated in infected urothelium relative to control
- Results files exist in the workflow output directory after the Snakemake run completes
- Regulated lipids satisfy the thresholds p < 0.05 and |log2(FC)| > 1
- Statistics are computed per ROI (urothelium, lamina propria, muscle) and grouped correctly against infected (n=4) vs control (n=4) metadata
- Volcano plot axes match log2 fold change (x) and -log10 p-value (y) with significant species highlighted by lipid class

## Limitations

- msiFlow does not include an annotation or interpretation step; lipid annotations must be made by mass matching to databases such as LipidMaps
- Complete cell-specificity is limited by the use of consecutive tissue sections and lateral diffusion of lipids during MSI sample preparation
- Group comparison requires prior segmentation/ROI annotation; it cannot be run directly on raw imaging data

## Evidence
