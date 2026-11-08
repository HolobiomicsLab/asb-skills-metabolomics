---
name: volcano-plot-visualization-of-regulated-lipids
description: Use when a segmented, group-annotated MALDI MSI pixel-intensity table
  exists and you need to visualize which lipid species are significantly up- or down-regulated
  between two groups (e.g. UPEC-infected vs. control) within specific tissue ROIs.
when_to_use_negative:
- Input is raw imzML/Bruker/Waters data that has not been pre-processed and segmented
  (run msi_preprocessing_flow and msi_segmentation_flow first)
- The analysis goal is lipid annotation/identification rather than differential regulation
  visualization — msiFlow explicitly does not include an annotation or interpretation
  step
- 'Comparing more than two groups: region_group_analysis_flow performs statistical
  analysis of molecular changes in ROIs between exactly two groups'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Provides the region_group_analysis_flow Snakemake workflow that performs pixel-wise
    statistical testing of mean lipid intensities between two groups within each ROI,
    yielding the log2 FC and p-value table used for volcano plotting
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Command-line workflow engine used to execute region_group_analysis_flow with
    a configfile and parallel cores
- name: pandas
  role: Loads the segmented pixel-intensity table and aggregates per-lipid statistics
    into the regulated-lipid table
- name: matplotlib
  role: Generates the volcano plots (log2 fold change vs. -log10 p-value) per ROI
    with significant species highlighted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:89f46f38905535b3d5520f1eb2ecd43c96dc91aa043b97c659f21c585415e8da
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
  - Snakemake
  - matplotlib
  - msiFlow
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Volcano-plot visualization of regulated lipids

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates volcano plots (log2 fold change vs. -log10 p-value) of lipids regulated between groups within mass spectrometry imaging ROIs, using msiFlow's region_group_analysis_flow statistical output to highlight significant lipid classes such as TG, DG, PC, PE and ST in infected vs. control tissue.

## When to use

Use when a segmented, group-annotated MALDI MSI pixel-intensity table exists and you need to visualize which lipid species are significantly up- or down-regulated between two groups (e.g. UPEC-infected vs. control) within specific tissue ROIs.

## When NOT to use

- Input is raw imzML/Bruker/Waters data that has not been pre-processed and segmented (run msi_preprocessing_flow and msi_segmentation_flow first)
- The analysis goal is lipid annotation/identification rather than differential regulation visualization — msiFlow explicitly does not include an annotation or interpretation step
- Comparing more than two groups: region_group_analysis_flow performs statistical analysis of molecular changes in ROIs between exactly two groups

## Inputs

- Segmented, ROI-annotated MSI pixel-intensity table from msi_segmentation_flow (loaded with pandas)
- Group metadata mapping samples to infected (n=4) and control (n=4) groups
- ROI labels mapping pixels to urothelium, lamina propria and muscle regions
- region_group_analysis_flow config file (YAML) specifying the two groups and ROIs

## Outputs

- Per-lipid table of log2 fold-changes and p-values for urothelium, lamina propria and muscle, ranked by significance and effect direction
- Volcano plot (log2 fold change vs. -log10 p-value) per ROI with significantly regulated lipid species highlighted by lipid class (TG, DG, PC, PE, ST)
- Results files in the workflow output directory

## How to apply

Load the segmented, ROI-annotated MSI pixel-intensity table produced by msi_segmentation_flow with pandas and attach group metadata (infected n=4 vs. control n=4) plus ROI labels (urothelium, lamina propria, muscle). Run msiFlow's region_group_analysis_flow via Snakemake (`snakemake --snakefile region_group_analysis_flow/Snakefile --cores <n> --configfile <config>`) to perform pixel-wise statistical testing of mean lipid intensities between the two groups within each ROI. Aggregate per-lipid log2 fold-changes and p-values into a regulated-lipid table, then plot log2 fold change against -log10 p-value per ROI with matplotlib, coloring/annotating significant species (p < 0.05, |log2(FC)| > 1) and their lipid classes. The rationale is that ROI-stratified statistics localize regulation: in the UPEC study the strongest alterations appeared in the infected urothelium, where TG, DG and PC were upregulated while PE and ST were downregulated.

## Related tools

- **msiFlow** (Provides the region_group_analysis_flow Snakemake workflow that performs pixel-wise statistical testing of mean lipid intensities between two groups within each ROI, yielding the log2 FC and p-value table used for volcano plotting) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Command-line workflow engine used to execute region_group_analysis_flow with a configfile and parallel cores)
- **pandas** (Loads the segmented pixel-intensity table and aggregates per-lipid statistics into the regulated-lipid table)
- **matplotlib** (Generates the volcano plots (log2 fold change vs. -log10 p-value) per ROI with significant species highlighted)

## Examples

```
snakemake --snakefile region_group_analysis_flow/Snakefile --cores 8 --configfile config.yaml
```

## Evaluation signals

- Volcano plots reproduce the reported direction of regulation: TG/DG/PC species appear right-shifted (up-regulated) and PE/ST species left-shifted (down-regulated) in infected urothelium relative to control
- Significant species satisfy the dual cutoff p < 0.05 AND |log2(FC)| > 1; species failing either cutoff are not highlighted as significant
- One regulated-lipid table and one volcano plot exist per ROI (urothelium, lamina propria, muscle), and results files are present in the workflow output directory
- Per-lipid log2 FC and p-value columns are complete and rankings by significance/effect direction are stable when the workflow is re-run reproducibly

## Limitations

- msiFlow does not include an annotation or interpretation step; tentative lipid annotations for interesting candidates must be made separately through mass matching to databases such as LipidMaps
- Volcano plots summarize mean ROI-level statistics and do not resolve cell-level specificity, which is limited by use of consecutive tissue sections and lateral diffusion of lipids during sample preparation
- Requires a completed segmentation/ROI annotation upstream; without valid ROI and group metadata the per-ROI statistics are undefined

## Evidence
