---
name: isobutane-ci-fragmentation-evaluation
description: Decision support for choosing a GC-MS ionization configuration (isobutane
  CI vs methane CI vs EI at 70 or 15 eV) when metabolites fragment excessively or
  the [M+H]+ isotopic envelope is too weak for isotopologue quantification.
when_to_use_negative:
- The analytes are not volatile small metabolites amenable to GC after methoxyamination/silylation
  derivatization — CI-isobutane protonation behavior may not transfer
- The goal is isotope ratio fidelity measurement at high resolution rather than choosing
  an ionization reagent gas; the article addresses reagent-gas selection, not HRMS
  isotope-ratio calibration
- Data are already processed isotopologue tables (e.g. isoSCAN output); this skill
  evaluates acquisition ionization, not downstream isotopologue quantification
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0157
tools:
- name: Agilent 7890A GC / 7000 QqQ mass spectrometer
  role: Acquire LRMS spectra under CI-isobutane, CI-methane, and EI modes for fragmentation
    comparison in SIM and narrow-window scan modes
- name: Agilent 7890B GC / 7250 QTOF mass spectrometer
  role: Acquire high-resolution full-scan spectra to confirm that isobutane CI concentrates
    abundance in the [M+H]+ envelope
- name: isoSCAN
  role: Downstream R tool that processes the validated GC-CI-MS data to automatically
    quantify all isotopologues once the ionization configuration is chosen
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:dbd438c3575e52648f470eaff4c7458081146861cd3675327927094def14bfe4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - Agilent 7890A GC / 7000 QqQ mass spectrometer
  - Agilent 7890B GC / 7250 QTOF mass spectrometer
  - GC-(q)TOF
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 26
---

# isobutane-ci-fragmentation-evaluation

## Summary

A GC-MS method-validation skill for comparing chemical ionization reagent gases and ionization modes by assessing where metabolite abundance concentrates in the isotopic envelope, to select the configuration that best preserves intact protonated molecular ions [M+H]+ for stable-isotope tracing.

## When to use

Decision support for choosing a GC-MS ionization configuration (isobutane CI vs methane CI vs EI at 70 or 15 eV) when metabolites fragment excessively or the [M+H]+ isotopic envelope is too weak for isotopologue quantification.

## When NOT to use

- The analytes are not volatile small metabolites amenable to GC after methoxyamination/silylation derivatization — CI-isobutane protonation behavior may not transfer
- The goal is isotope ratio fidelity measurement at high resolution rather than choosing an ionization reagent gas; the article addresses reagent-gas selection, not HRMS isotope-ratio calibration
- Data are already processed isotopologue tables (e.g. isoSCAN output); this skill evaluates acquisition ionization, not downstream isotopologue quantification

## Inputs

- Derivatized central carbon metabolite extracts (methoxyamination + MSTFA/1% TMCS silylation) from labeled (U-13C-glucose) and non-labeled cell cultures
- GC-MS raw spectra acquired under CI-isobutane, CI-methane, EI 70 eV, and EI 15 eV on instruments such as an Agilent 7890A GC / 7000 QqQ or 7890B GC / 7250 QTOF
- A reference metabolite (e.g. lactate) with known isotopologue pattern for spectral comparison

## Outputs

- Per-metabolite mass spectra annotated with the location of metabolite abundance ([M+H]+ isotopic envelope vs fragment ions) per ionization condition
- [M+H]+ intensity comparison (isobutane vs methane) across glycolytic and TCA cycle intermediates
- A ranked recommendation of ionization configuration (reagent gas and EI energy) for isotopologue detection

## How to apply

Acquire GC-MS spectra of the same derivatized metabolite set (e.g. methoxyamination plus MSTFA/1% TMCS silylation) under each ionization condition: CI with isobutane, CI with methane, EI at 70 eV, and low-energy EI at 15 eV, using labeled (e.g. U-13C-glucose) and non-labeled cell-culture samples. For each condition, inspect the mass spectra of a reference metabolite such as lactate and quantify where metabolite abundance sits: with isobutane CI the abundance concentrates in the [M+H]+ isotopic envelope, whereas methane CI induces unwanted fragmentation of the [M+H]+ ion and EI causes extensive structural fragmentation. Compute the average [M+H]+ intensity ratio between isobutane and methane across glycolytic and TCA cycle intermediates; the article reports >5-fold higher intensity with isobutane. Prefer the reagent gas/mode yielding intact protonated molecular ions with minimal fragment ions, since that maximizes isotopologue coverage; use lactate in labeled vs non-labeled cultures as the sanity-check reference metabolite.

## Related tools

- **Agilent 7890A GC / 7000 QqQ mass spectrometer** (Acquire LRMS spectra under CI-isobutane, CI-methane, and EI modes for fragmentation comparison in SIM and narrow-window scan modes)
- **Agilent 7890B GC / 7250 QTOF mass spectrometer** (Acquire high-resolution full-scan spectra to confirm that isobutane CI concentrates abundance in the [M+H]+ envelope)
- **isoSCAN** (Downstream R tool that processes the validated GC-CI-MS data to automatically quantify all isotopologues once the ionization configuration is chosen) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
install.packages("devtools", dependencies=TRUE); library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- Isobutane CI spectra should show metabolite abundance concentrated in the [M+H]+ isotopic envelope, while methane CI spectra show fragmented [M+H]+ ions
- Average [M+H]+ ion intensity with isobutane should exceed methane by >5-fold across all glycolytic and TCA cycle intermediates
- EI at 70 eV should show extensive fragmentation of compound structures in both non-labeled and labeled samples, and EI at 15 eV should still fragment the molecular ion more than CI-isobutane
- Isotopic enrichments of the reference metabolite lactate should be detectable in U-13C-glucose labeled vs non-labeled cultures under CI-isobutane
- The chosen configuration should enable precise detection of isotopologues of central carbon metabolism intermediates on both LR and HR MS

## Limitations

- EI even at reduced 15 eV energy produces greater fragmentation of the molecular ion than CI-isobutane, so EI-based configurations cannot simply be rescued by lowering energy
- Methane CI fragments the [M+H]+ ion, so reagent gas choice is not interchangeable — evaluation must be redone if switching gases
- Performance statements are grounded in central carbon metabolism intermediates (glycolysis, TCA cycle) and lactate as reference; other metabolite classes were not evaluated here

## Evidence

- [results] Isobutane as reagent gas concentrated metabolite abundance in the [M+H]+ isotopic envelope, while methane induced unwanted fragmentation of the [M+H]+ ion: [evidence span withheld — anchor status: not_found]
- [results] >5-fold [M+H]+ intensity advantage of isobutane over methane: [evidence span withheld — anchor status: not_found]
- [results] EI at 70 eV causes extensive fragmentation: [evidence span withheld — anchor status: not_found]
- [results] Low-energy EI 15 eV still fragments more than CI-isobutane: [evidence span withheld — anchor status: not_found]
- [intro] Isobutane yields intact protonated molecular ions and is the ideal ionization gas: [evidence span withheld — anchor status: not_found]
- [readme] README confirms isoSCAN processes GC-CI-MS stable isotope labelling data: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
