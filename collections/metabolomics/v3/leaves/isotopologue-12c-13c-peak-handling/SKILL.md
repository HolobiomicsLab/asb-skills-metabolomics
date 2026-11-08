---
name: isotopologue-12c-13c-peak-handling
description: Use when grouping co-eluting MS1 peaks into CSA clusters from IDSL.IPA
  peaklists/peak_alignment directories and you must decide whether an m/z-shifted
  neighbor is a genuine 12C/13C isotopologue of the seed peak.
when_to_use_negative:
- The goal is peak detection or EIC extraction itself — use IDSL.IPA (this step assumes
  peaklists and peak_alignment already exist).
- The data are MS2-based (DDA/DIA) fragmentation spectra; IDSL.CSA supports those
  modes, but 12C/13C isotopologue grouping is specific to MS1-only CSA cluster aggregation.
- m/z spacing does not correspond to a plausible isotopologue of the seed (e.g. adduct
  or in-source fragment spacing); these should be handled through CSA adduct-annotation
  logic rather than isotopologue handling.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.CSA
  role: Implements the CSA clustering that includes the 12C/13C isotopologue handling
    of correlated MS1 neighbor peaks; configured via CSA parameter spreadsheet.
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream extraction of the peaklists, peak_alignment table, and individual
    EICs whose smoothed curves are used to flag isotopologue neighbors.
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:fc14433173518d3a3025fc3b29d4c1bb69c1a84a248ce95957e4c72ea9513a58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# isotopologue-12c-13c-peak-handling

## Summary

Within IDSL.CSA composite-spectra aggregation, correlating MS1 neighbor peaks are classified and retained as expected 12C/13C isotopologue pairs based on their m/z spacing relative to the seed peak, rather than being discarded as unexplained co-eluting ions.

## When to use

Use when grouping co-eluting MS1 peaks into CSA clusters from IDSL.IPA peaklists/peak_alignment directories and you must decide whether an m/z-shifted neighbor is a genuine 12C/13C isotopologue of the seed peak.

## When NOT to use

- The goal is peak detection or EIC extraction itself — use IDSL.IPA (this step assumes peaklists and peak_alignment already exist).
- The data are MS2-based (DDA/DIA) fragmentation spectra; IDSL.CSA supports those modes, but 12C/13C isotopologue grouping is specific to MS1-only CSA cluster aggregation.
- m/z spacing does not correspond to a plausible isotopologue of the seed (e.g. adduct or in-source fragment spacing); these should be handled through CSA adduct-annotation logic rather than isotopologue handling.

## Inputs

- IDSL.IPA 'peaklists' directory
- IDSL.IPA 'peak_alignment' directory
- Aligned peak table (m/z-RT) with S/N values
- CSA parameter spreadsheet (e.g. CSA_parameters.xlsx) configuring m/z tolerance and CSA0005/CSA0008/CSA0009/CSA0011 paths

## Outputs

- CSA cluster membership per seed peak recorded in aligned_spectra_table.csv
- Composite spectra (.msp) files in CSA_MSP containing retained 12C/13C isotopologue ions
- peak_alignment_subset tables for major ions in each CSA cluster

## How to apply

After selecting a seed peak from the IDSL.IPA aligned peak table by S/N ranking and collecting co-eluting neighbors whose LOESS+cubic-spline smoothed EICs show Pearson correlation >= 0.98, examine each correlated neighbor's m/z difference from the seed. A neighbor whose m/z spacing matches the expected 1.003355 Da (or the appropriate isotopologue spacing within the CSA parameter spreadsheet's m/z tolerance) is handled as a 12C/13C isotopologue pair, and the minimum ion count and m/z-range filters are applied to these pairs before they are aggregated into the CSA cluster written to aligned_spectra_table.csv. This keeps isotopologues inside the composite spectrum instead of leaving them as orphan peaks, which is why IDSL.CSA's CSA libraries can be annotated downstream without MS2 data.

## Related tools

- **IDSL.CSA** (Implements the CSA clustering that includes the 12C/13C isotopologue handling of correlated MS1 neighbor peaks; configured via CSA parameter spreadsheet.) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream extraction of the peaklists, peak_alignment table, and individual EICs whose smoothed curves are used to flag isotopologue neighbors.) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Every CSA cluster containing an isotopologue neighbor should have that neighbor's m/z within the expected 12C/13C spacing (≈1.003355 Da within the parameter spreadsheet's tolerance) of the seed peak.
- Correlated isotopologue neighbors must additionally satisfy the CSA minimum ion count and m/z range filters before entering aligned_spectra_table.csv.
- A visible validation is that the case-study run reproduces the reported 255 peaks, 153 neighbors, and 42 clusters on the pooled-standards test file.
- CSA_MSP output should show the isotopologue as an included peak with an intensity and RT consistent with the seed, not as a separate unmatched entry.
- Downstream IDSL.FSA searching of the .msp should return a hit whose molecular formula charge state is consistent with the number of 13C isotopologue ions retained.

## Limitations

- Isotopologue handling depends on the CSA parameter spreadsheet's m/z tolerance and S/N seed ranking; poor settings can miss or falsely group isotopologue pairs.
- The method is calibrated for LC/HRMS MS1-only data from instruments such as Thermo Q Exactive HF hybrid Orbitrap; very low-resolution or GC/HRMS data may not resolve 12C/13C spacing cleanly.
- Isotopologue assignment is heuristic — it does not replace a full isotope-pattern scoring tool and may misclassify co-eluting ions with coincidental ~1.0034 Da spacing.

## Evidence

- [methods] Applying minimum ion count and m/z range filters to the correlated neighbors and handle 12C/13C isotopologue pairs according to their expected m/z spacing.: [evidence span withheld — anchor status: not_found]
- [abstract] The CSA deconvolution mechanism extracts EICs for candidate co-eluting ions and computes Pearson correlations among the grouped peaks' EICs after LOESS smoothing followed by cubic spline interpolation, grouping correlating MS1 ions into CSA clusters.: [evidence span withheld — anchor status: not_found]
