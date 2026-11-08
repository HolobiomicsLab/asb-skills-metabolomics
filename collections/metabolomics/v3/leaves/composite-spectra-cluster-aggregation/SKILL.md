---
name: composite-spectra-cluster-aggregation
description: Use when you have IDSL.IPA-generated 'peaklists' and 'peak_alignment'
  outputs from LC/HRMS MS1-only data and need to group co-eluting, correlating ions
  into CSA clusters for downstream composite spectra library creation and chemical
  annotation.
when_to_use_negative:
- Your goal is to annotate peaks from true MS2 fragmentation spectra (DDA) — use the
  DDA analysis mode of IDSL.CSA or standard MS2 spectral library search instead of
  MS1-only composite spectra aggregation.
- Input is already a feature table or spectral library without per-peak EIC chromatograms;
  the aggregation requires IDSL.IPA-generated peaklists and aligned peak tables as
  prerequisites.
- You need only single-peak chemical annotation (e.g. isotopologue/adduct flagging
  without grouped spectra) — existing MS1 grouping approaches limited to flagging
  peaks may suffice.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IDSL.CSA
  role: Performs the composite spectra deconvolution and CSA cluster aggregation on
    the peaklists and aligned peak tables.
  repo: github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: 'Upstream prerequisite: generates the peaklists and peak_alignment directories
    consumed by the CSA workflow.'
  repo: github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: 'Downstream consumer: searches the resulting composite spectra (.msp) libraries
    for chemical annotation.'
  repo: github.com/idslme/IDSL.FSA
- name: R
  role: Runtime environment; the workflow is invoked as IDSL.CSA_workflow() from an
    installed R package.
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:d2e5c8df40f583c00cc9c1165ed9ce4c8beab6114cc3080692cdbcfc054000bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# composite-spectra-cluster-aggregation

## Summary

Aggregates correlating co-eluting MS1 chromatographic peaks (isotopologues, adducts, in-source fragments) into composite spectra (CSA) clusters using IDSL.CSA, enabling composite mass spectra libraries from LC/HRMS data even when MS2 spectra are unavailable.

## When to use

Use when you have IDSL.IPA-generated 'peaklists' and 'peak_alignment' outputs from LC/HRMS MS1-only data and need to group co-eluting, correlating ions into CSA clusters for downstream composite spectra library creation and chemical annotation.

## When NOT to use

- Your goal is to annotate peaks from true MS2 fragmentation spectra (DDA) — use the DDA analysis mode of IDSL.CSA or standard MS2 spectral library search instead of MS1-only composite spectra aggregation.
- Input is already a feature table or spectral library without per-peak EIC chromatograms; the aggregation requires IDSL.IPA-generated peaklists and aligned peak tables as prerequisites.
- You need only single-peak chemical annotation (e.g. isotopologue/adduct flagging without grouped spectra) — existing MS1 grouping approaches limited to flagging peaks may suffice.

## Inputs

- IDSL.IPA 'peaklists' directory (per-sample and aggregated peaklists of m/z-RT peaks)
- IDSL.IPA 'peak_alignment' directory (aligned peak table)
- IDSL.CSA parameter spreadsheet (e.g. CSA_parameters.xlsx) with CSA workflow parameters (CSA0004 threads, CSA0005 HRMS data location, CSA0008 peaklists address, CSA0009 peak_alignment address, CSA0011 output address)
- MS1-level HRMS raw data files (mzXML, mzML, netCDF) referenced by CSA0005

## Outputs

- aligned_spectra_table.csv with CSA aggregation results on the aligned table
- CSA_MSP directory containing .msp composite spectra files
- CSA_adduct_annotation directory with peaklists carrying potential adduct information
- peak_alignment_subset directory with subsets of aligned peak tables for major ions in each CSA cluster
- Batch untargeted aligned EIC figures for CSA analysis

## How to apply

Load the IDSL.IPA-generated 'peaklists' and 'peak_alignment' directories together with the IDSL.CSA parameter spreadsheet as the module configuration, selecting CSA-specific parameters (e.g. PARAM0001=Yes, CSA0005/0008/0009/0011 addresses). Select seed peaks from the aligned peak table by signal-to-noise (S/N) ranking; for each seed, gather co-eluting candidates within a retention-time window and compute Pearson correlations between their extracted ion chromatograms (EICs) after LOESS smoothing followed by cubic spline interpolation, retaining neighbors with correlation ≥ 0.98. Apply minimum ion-count and m/z-range filters and handle 12C/13C isotopologue pairs according to their expected m/z spacing before aggregating qualifying neighbors into CSA clusters. Write aggregation results to aligned_spectra_table.csv; outputs also include .msp spectra (CSA_MSP), potential adduct annotations, and peak_alignment_subset tables for major ions in each cluster.

## Related tools

- **IDSL.CSA** (Performs the composite spectra deconvolution and CSA cluster aggregation on the peaklists and aligned peak tables.) — github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream prerequisite: generates the peaklists and peak_alignment directories consumed by the CSA workflow.) — github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream consumer: searches the resulting composite spectra (.msp) libraries for chemical annotation.) — github.com/idslme/IDSL.FSA
- **R** (Runtime environment; the workflow is invoked as IDSL.CSA_workflow() from an installed R package.) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- On the pooled-standards test file the module must reproduce 255 peaks, 153 neighbors, and 42 clusters.
- aligned_spectra_table.csv exists and lists CSA aggregation information consistent with the aligned peak table.
- CSA_MSP directory contains .msp files whose spectra reflect grouped co-eluting MS1 ions rather than single MS2 spectra.
- peak_alignment_subset tables contain only major ions belonging to each CSA cluster.
- Batch EIC figures are generated for DIA/CSA analyses and are visually consistent with the correlation-based grouping.

## Limitations

- Requires upstream IDSL.IPA processing; raw files cannot be fed directly into the CSA workflow.
- Correlation threshold (0.98) and RT-window, minimum ion count, and m/z-range parameters must be tuned per dataset; default values in the parameter spreadsheet may not fit all acquisition modes.
- Composite spectra are built from MS1-only signals; annotation quality depends on the assumption that co-eluting correlating ions belong to the same compound, which may fail for isobaric co-elution.
- Parallel processing works in Windows and Linux environments but thread count (CSA0004) must be adjusted to available compute.

## Evidence

- [methods] seed peak S/N ranking and correlation threshold: [evidence span withheld — anchor status: not_found]
- [abstract] grouping correlating MS1 ions into CSA clusters: [evidence span withheld — anchor status: not_found]
- [readme] workflow prerequisite: IDSL.IPA output directories: [evidence span withheld — anchor status: not_found]
- [readme] output directory layout: [evidence span withheld — anchor status: not_found]
- [readme] parameter spreadsheet configuration: [evidence span withheld — anchor status: not_found]
