---
name: gc-ci-ms-isotopologue-configuration-comparison
description: Use when selecting or benchmarking a GC-CI-MS platform for stable-isotope
  tracing, e.g. deciding between SIM-mode QqQ and full-scan high-resolution qTOF/Orbitrap
  for isotopologue quantification.
when_to_use_negative:
- EI (70 eV or 15 eV) spectra are the input — EI causes extensive fragmentation of
  molecular ions, so the LR/HR comparison under CI conditions does not apply.
- The analytes are outside central carbon metabolism intermediates for which the LR/HR
  performance statements were established.
- Isotope-ratio fidelity at high mass resolution is the primary metric — the article
  reports HRMS has inferior isotope-ratio fidelity versus LRMS.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: isoSCAN
  role: R package that automatically quantifies all isotopologues from GC-CI-MS data
    of both LRMS and HRMS configurations
  repo: https://github.com/jcapelladesto/isoSCAN
- name: Agilent 7890A GC / 7000 QqQ MS
  role: Low-resolution instrument used for isotopologue determination in SIM and narrow-window
    NS acquisition modes
- name: Agilent 7890B GC / 7250 QTOF MS
  role: High-resolution full-scan instrument compared against the QqQ configurations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:8e11e6919c8ee99aac4733c5463844d2522b121158a00509ee0e2b320e2e0b75
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - Agilent 7890A GC / 7000 QqQ MS
  - Agilent 7890B GC / 7250 QTOF MS
  - GC-(q)TOF
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# gc-ci-ms-isotopologue-configuration-comparison

## Summary

Compares LR (QqQ SIM/narrow-window scan) vs HR (GC-qTOF, GC-Orbitrap) GC-CI-MS configurations for detecting isotopologues of central carbon metabolites under isobutane chemical ionization, using isoSCAN to quantify all isotopologues.

## When to use

Use when selecting or benchmarking a GC-CI-MS platform for stable-isotope tracing, e.g. deciding between SIM-mode QqQ and full-scan high-resolution qTOF/Orbitrap for isotopologue quantification.

## When NOT to use

- EI (70 eV or 15 eV) spectra are the input — EI causes extensive fragmentation of molecular ions, so the LR/HR comparison under CI conditions does not apply.
- The analytes are outside central carbon metabolism intermediates for which the LR/HR performance statements were established.
- Isotope-ratio fidelity at high mass resolution is the primary metric — the article reports HRMS has inferior isotope-ratio fidelity versus LRMS.

## Inputs

- GC-CI-MS acquisition data from Agilent 7000 QqQ (SIM and ~10 Da NS scan modes)
- Full-scan HR data from Agilent 7250 GC-qTOF or GC-Orbitrap
- U-13C-glucose or U-13C-arginine labeled cell-culture metabolite extracts
- Isobutane CI chromatograms with [M+H]+ isotopic envelopes

## Outputs

- Configuration comparison table (instrument vs reagent gas, resolution, isotopologue coverage, performance notes)
- CSV summary of LRMS vs HRMS isotopologue-detection performance
- isoSCAN-quantified isotopologue enrichments from LR and HR datasets

## How to apply

Tabulate each configuration — Agilent 7000 QqQ in SIM and narrow-window (~10 Da) NS modes, Agilent 7250 QTOF, and GC-Orbitrap — against isobutane CI reagent gas, mass resolution, and isotopologue coverage of glycolysis/TCA intermediates. Rationale: isobutane concentrates metabolite abundance in the intact [M+H]+ ion (on average >5-fold more intense than methane CI), giving clean isotopic envelopes that both LR and HR instruments can measure. Judge an instrument by its sensitivity (signal intensity in full scan) and isotope-ratio fidelity, since the article reports latest-generation HR instruments in full scan show sensitivity similar to quadrupole analyzers but isotope ratio fidelity is superior in LRMS than HRMS. Use isoSCAN (R package) to automatically process the resulting GC-CI-MS data from both LRMS and HRMS runs and export a configuration-vs-performance table (e.g. CSV).

## Related tools

- **isoSCAN** (R package that automatically quantifies all isotopologues from GC-CI-MS data of both LRMS and HRMS configurations) — https://github.com/jcapelladesto/isoSCAN
- **Agilent 7890A GC / 7000 QqQ MS** (Low-resolution instrument used for isotopologue determination in SIM and narrow-window NS acquisition modes)
- **Agilent 7890B GC / 7250 QTOF MS** (High-resolution full-scan instrument compared against the QqQ configurations)

## Examples

```
library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- All four tested configurations (QqQ SIM, QqQ NS ~10 Da, GC-qTOF, GC-Orbitrap) appear as rows in the comparison table.
- The reagent gas column reads isobutane and a >5-fold [M+H]+ intensity advantage over methane is reported.
- Both LR and HR rows show isotopologue coverage of glycolysis and TCA cycle intermediates.
- Output CSV has clear column headers covering ionization reagent gas, mass resolution, isotopologue coverage, and performance notes.
- Quantified isotopologues are produced by isoSCAN rather than manual peak picking.

## Limitations

- Full-scan HR instruments (GC-qTOF, GC-Orbitrap) show similar sensitivity to quadrupole-based analyzers, but isotope-ratio fidelity is superior in LRMS than HRMS.
- Methane CI induces unwanted fragmentation of the [M+H]+ ion; EI at 70 eV and 15 eV causes greater fragmentation than CI-isobutane.
- Previously published cell-culture protocols were referenced instead of fully described; no software version given for isoSCAN in the article.

## Evidence

- [abstract] LR vs HR instrument classes for isobutane CI: [evidence span withheld — anchor status: not_found]
- [intro] isoSCAN processes both LR and HR GC-CI-MS data: [evidence span withheld — anchor status: not_found]
- [results] isobutane advantage over methane quantified: [evidence span withheld — anchor status: not_found]
- [methods] QqQ SIM and narrow-window NS acquisition modes: [evidence span withheld — anchor status: not_found]
- [methods] HR instruments compared against QqQ: [evidence span withheld — anchor status: not_found]
- [full_text] LRMS vs HRMS isotope-ratio fidelity trade-off: [evidence span withheld — anchor status: not_found]
- [readme] README installation instructions: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
