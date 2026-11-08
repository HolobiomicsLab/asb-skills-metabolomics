---
name: coeluting-ion-correlation-grouping
description: Use when you have IDSL.IPA-generated 'peaklists' and 'peak_alignment'
  directories plus MS1-level HRMS data and need to group co-eluting MS1 ions (isotopologues,
  adducts, in-source fragments) into composite spectra clusters for chemical annotation
  without MS2 data.
when_to_use_negative:
- When MS/MS fragmentation spectra are already available and the goal is conventional
  MS2 spectral-library annotation rather than MS1-only composite spectra reconstruction.
- When chromatographic peak detection/alignment has not yet been performed — the skill
  requires IDSL.IPA 'peaklists' and 'peak_alignment' outputs as prerequisites; raw
  mzXML/mzML/netCDF alone is insufficient.
- When the analysis is DDA or DIA (MS^E, AIF, SWATH-MS) deconvolution rather than
  MS1-only Composite Spectra Analysis — select a different workflow in PARAM0001.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.CSA
  role: R package that performs the seed-peak selection, RT-window neighbor gathering,
    EIC Pearson-correlation grouping and CSA cluster aggregation via IDSL.CSA_workflow
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak detection and alignment that supplies the 'peaklists' and 'peak_alignment'
    directories consumed by the CSA grouping step
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Downstream companion package for mass spectral similarity searches against
    the generated CSA .msp libraries
  repo: https://github.com/idslme/IDSL.FSA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:bf72358e1f081382219459fb8d16546f49cae2a1b822e1bdf5df46e50e38af9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# coeluting-ion-correlation-grouping

## Summary

Deconvolves MS1-only LC/HRMS data by grouping co-eluting ions into composite spectra (CSA) clusters: seed peaks are ranked by signal-to-noise, candidate neighbors within a retention-time window are correlated on LOESS+cubic-spline smoothed EICs (Pearson >= 0.98), and qualifying ions are aggregated into CSA clusters written to aligned_spectra_table.csv.

## When to use

Use when you have IDSL.IPA-generated 'peaklists' and 'peak_alignment' directories plus MS1-level HRMS data and need to group co-eluting MS1 ions (isotopologues, adducts, in-source fragments) into composite spectra clusters for chemical annotation without MS2 data.

## When NOT to use

- When MS/MS fragmentation spectra are already available and the goal is conventional MS2 spectral-library annotation rather than MS1-only composite spectra reconstruction.
- When chromatographic peak detection/alignment has not yet been performed — the skill requires IDSL.IPA 'peaklists' and 'peak_alignment' outputs as prerequisites; raw mzXML/mzML/netCDF alone is insufficient.
- When the analysis is DDA or DIA (MS^E, AIF, SWATH-MS) deconvolution rather than MS1-only Composite Spectra Analysis — select a different workflow in PARAM0001.

## Inputs

- MS1-level HRMS raw data (mzXML, mzML, netCDF)
- IDSL.IPA-generated 'peaklists' directory
- IDSL.IPA-generated 'peak_alignment' directory
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) with CSA0005 (MS1 HRMS data location), CSA0008 (peaklists address), CSA0009 (peak_alignment address), CSA0011 (output location)

## Outputs

- CSA_MSP directory with .msp composite spectra files
- CSA_adduct_annotation peaklists with potential adduct information
- peak_alignment_subset tables for the major ions in each CSA cluster
- aligned_spectra_table.csv with CSA aggregation results

## How to apply

Load the IDSL.IPA 'peaklists' and 'peak_alignment' directories together with the IDSL.CSA parameter spreadsheet (39 parameters in 5 sections) as the module configuration. Select seed peaks from the aligned peak table by signal-to-noise (S/N) ranking. For each seed peak, gather co-eluting candidate neighbors within the retention-time window, then compute Pearson correlations between the neighbors' extracted ion chromatograms after LOESS smoothing followed by cubic spline interpolation, retaining neighbors with correlation >= 0.98. Apply minimum-ion-count and m/z range filters, handle 12C/13C isotopologue pairs according to their expected m/z spacing, and aggregate qualifying neighbors into CSA clusters that are written to aligned_spectra_table.csv. Increase CSA0004 (processing threads) according to available compute; results are parsed under CSA0011.

## Related tools

- **IDSL.CSA** (R package that performs the seed-peak selection, RT-window neighbor gathering, EIC Pearson-correlation grouping and CSA cluster aggregation via IDSL.CSA_workflow) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream peak detection and alignment that supplies the 'peaklists' and 'peak_alignment' directories consumed by the CSA grouping step) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream companion package for mass spectral similarity searches against the generated CSA .msp libraries) — https://github.com/idslme/IDSL.FSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Reproduced cluster counts on the pooled-standards test file: 255 peaks, 153 neighbors, 42 clusters
- aligned_spectra_table.csv exists and contains rows for each CSA cluster aggregation on the aligned peak table
- Every retained neighbor's LOESS+cubic-spline smoothed EIC Pearson correlation is >= 0.98 with the seed peak
- Each CSA cluster satisfies the configured minimum ion count and m/z range filters, including correct 12C/13C isotopologue m/z spacing
- CSA_MSP output contains a .msp file whose spectra correspond to the major ions listed in peak_alignment_subset

## Limitations

- Retention-time grouping and EIC correlation depend on upstream IDSL.IPA peak detection quality; misaligned RTs degrade clustering.
- Correlation threshold (>= 0.98) after LOESS+cubic-spline smoothing may merge co-eluting but unrelated ions or split weakly correlated true fragments.
- Results depend on 39 spreadsheet parameters; no changelog was provided, so version drift could alter behavior.
- Applicability is for LC/HRMS untargeted metabolomics (also GC); the paper does not describe validation beyond pooled-standards and human blood endogenous metabolites.

## Evidence

- [results] The CSA deconvolution mechanism extracts EICs for candidate co-eluting ions and computes Pearson correlations among the grouped peaks' EICs after LOESS smoothing followed by cubic spline interpolation: [evidence span withheld — anchor status: not_found]
- [methods] download the [IDSL.CSA parameter spreadsheet] and select the parameters accordingly and then use this spreadsheet as the input for the IDSL.CSA workflow: [evidence span withheld — anchor status: not_found]
