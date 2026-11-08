---
name: quantitative-ion-intensity-interpretation
description: Use when GC-CI-MS stable-isotope labeling data must be scored quantitatively
  — e.g., comparing [M+H]+ intensity across reagent gases or judging isotopologue
  detectability on QqQ SIM, narrow-window scan, qTOF, or Orbitrap platforms.
when_to_use_negative:
- The spectra were acquired under EI (70 eV or 15 eV) rather than CI — fragmentation
  makes the [M+H]+-centered intensity interpretation invalid
- The analytes fall outside derivatizable central carbon metabolites (glycolysis,
  TCA cycle, polyamines) for which this GC-CI-MS evaluation was performed
- Only chromatographic separation quality is of interest — the article's GC advantage
  claim is about separation efficiency, not isotopologue intensity quantification
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: R package that automatically quantifies all isotopologues from GC-CI-MS data
    in both LRMS and HRMS labeling studies
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment for isoSCAN isotopologue quantification of ion intensity
    data
  repo: https://github.com/jcapelladesto/isoSCAN
- name: triple quadrupole (QqQ) MS
  role: Low-resolution acquisition platform providing SIM and narrow-window scan intensity
    data for isotopologue determination
- name: GC-(q)TOF
  role: High-resolution full-scan platform used to benchmark sensitivity and isotope
    ratio fidelity against QqQ
- name: GC-Orbitrap
  role: High-resolution full-scan platform compared in the configuration matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:cce318326032ad0326178d1be7feeb09b952713ef0b926eb23e1e0cdf0cf3bb7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - GC-(q)TOF
  - GC-Orbitrap
  - R
  - isoSCAN
  - triple quadrupole (QqQ) MS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# quantitative-ion-intensity-interpretation

## Summary

Interprets raw ion intensities from GC-CI-MS acquisitions to quantify isotopologue enrichment, concentrating interpretation on the [M+H]+ isotopic envelope produced by isobutane chemical ionization and comparing intensity across LRMS (QqQ SIM/NS) and HRMS (qTOF, Orbitrap) configurations.

## When to use

Use when GC-CI-MS stable-isotope labeling data must be scored quantitatively — e.g., comparing [M+H]+ intensity across reagent gases or judging isotopologue detectability on QqQ SIM, narrow-window scan, qTOF, or Orbitrap platforms.

## When NOT to use

- The spectra were acquired under EI (70 eV or 15 eV) rather than CI — fragmentation makes the [M+H]+-centered intensity interpretation invalid
- The analytes fall outside derivatizable central carbon metabolites (glycolysis, TCA cycle, polyamines) for which this GC-CI-MS evaluation was performed
- Only chromatographic separation quality is of interest — the article's GC advantage claim is about separation efficiency, not isotopologue intensity quantification

## Inputs

- GC-CI-MS raw ion intensity data from isobutane chemical ionization (QqQ SIM, QqQ narrow-window ~10 Da scan, GC-qTOF full scan, GC-Orbitrap full scan)
- List of central carbon metabolism intermediates (glycolytic pathway and TCA cycle) and their [M+H]+ target ions
- Labeled vs. non-labeled sample pairs (e.g., U-13C-glucose or U-13C-arginine cell culture extracts, methoxyamine/MSTFA+1% TMCS derivatized)

## Outputs

- Quantitative [M+H]+ intensity comparison table (e.g., isobutane vs. methane, with the reported >5-fold mean intensity advantage)
- Configuration comparison matrix (instrument, reagent gas, resolution, isotopologue coverage, isotope ratio fidelity) exportable as CSV
- isoSCAN-quantified isotopologue enrichment values for LRMS and HRMS datasets

## How to apply

Acquire data under isobutane chemical ionization, since isobutane concentrates metabolite abundance in the [M+H]+ isotopic envelope whereas methane induces fragmentation of [M+H]+ and EI (70 eV and 15 eV) fragments the molecular ion more extensively. Interpret intensity quantitatively by comparing the [M+H]+ ion response across reagent gases — the article reports it was, on average, >5-fold more intense with isobutane than methane for glycolytic and TCA cycle intermediates. Tabulate tested configurations (QqQ in SIM and narrow-window ~10 Da scan mode, GC-qTOF, GC-Orbitrap) with columns for reagent gas, resolution, and isotopologue coverage, populating rows only with reported values. Weigh isotope ratio fidelity separately from sensitivity: HR full-scan instruments reach similar sensitivity to quadrupole analyzers, but isotope ratio fidelity is superior in LRMS. Process both LRMS and HRMS datasets through the isoSCAN R package, which automatically quantifies all isotopologues.

## Related tools

- **isoSCAN** (R package that automatically quantifies all isotopologues from GC-CI-MS data in both LRMS and HRMS labeling studies) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment for isoSCAN isotopologue quantification of ion intensity data) — https://github.com/jcapelladesto/isoSCAN
- **triple quadrupole (QqQ) MS** (Low-resolution acquisition platform providing SIM and narrow-window scan intensity data for isotopologue determination)
- **GC-(q)TOF** (High-resolution full-scan platform used to benchmark sensitivity and isotope ratio fidelity against QqQ)
- **GC-Orbitrap** (High-resolution full-scan platform compared in the configuration matrix)

## Examples

```
devtools::install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- The [M+H]+ intensity ratio between isobutane-CI and methane-CI spectra reproduces the reported >5-fold mean advantage for glycolytic and TCA intermediates
- Abundance of each metabolite is concentrated in the [M+H]+ isotopic envelope with minimal fragmentation, unlike methane-CI or EI spectra
- Isotopologue enrichments quantified independently from QqQ SIM/NS data and HR full-scan data are concordant for the same samples
- Isotope ratio fidelity checks are better on the LRMS (QqQ) channel than on HRMS channels, matching the article's reported performance ranking
- isoSCAN output tables cover the same isotopologue counts across LRMS and HRMS inputs for central carbon metabolites

## Limitations

- HR full-scan instruments (qTOF, Orbitrap) show similar sensitivity to quadrupole analyzers but superior isotope ratio fidelity is achieved on LRMS — do not assume HRMS improves isotope ratios
- Low-energy EI at 15 eV still fragments the molecular ion more than CI-isobutane, so EI-based intensity interpretation underestimates intact [M+H]+ abundance
- Methane reagent gas causes unwanted fragmentation of the [M+H]+ ion, invalidating envelope-centered quantification
- Cell culturing protocols were referenced to prior publications rather than fully described, limiting reproducibility of upstream sample preparation

## Evidence

- [results] the [M+H]+ ion was, on average, >5-fold more intense using isobutane than methane (Figure 1C): [evidence span withheld — anchor status: not_found]
- [results] the use of isobutane as reagent gas produced mass spectra in which the abundance of a metabolite is typically concentrated in its [M+H]+ isotopic envelop: [evidence span withheld — anchor status: not_found]
- [abstract] Isobutane as reagent gas in combination with both low-resolution (LR) and high-resolution (HR) MS showed the best performance, enabling precise detection of isotopologues: [evidence span withheld — anchor status: not_found]
- [intro] an R-package called isoSCAN capable of processing GC- CI MS data from LRMS and HRMS stable-isotope labeling studies: [evidence span withheld — anchor status: not_found]
