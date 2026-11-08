---
name: lrms-hrms-resolution-coverage-interpretation
description: Use when choosing between or interpreting results from LRMS (QqQ SIM/NS)
  and HRMS (qTOF, GC-Orbitrap) acquisition for stable-isotope tracing, or when tabulating
  instrument-specific isotopologue coverage and isotope ratio fidelity.
when_to_use_negative:
- When the data were acquired with EI (70 eV or 15 eV) instead of CI — extensive molecular-ion
  fragmentation makes the [M+H]+ envelope comparison invalid
- When comparing against methane-CI spectra — methane induces unwanted [M+H]+ fragmentation,
  so intensity and coverage comparisons would be confounded
- When the study target is outside GC-amenable central carbon metabolites (e.g. un-derivatized
  LC-MS analytes) — GC separation efficiency is the stated basis of the advantage
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: R package that automatically quantifies all isotopologues from GC-CI-MS data
    of both LRMS and HRMS labeling studies, enabling cross-instrument comparison
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment for installing and running isoSCAN on the QqQ and HR full-scan
    datasets
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:c45cb6e814718f73038c665715af0b5def604bbc17fd9b8cdec780ce8d0a8a0a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - GC-(q)TOF
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# lrms-hrms-resolution-coverage-interpretation

## Summary

A comparison framework for judging how low-resolution (Agilent 7000 QqQ in SIM or ~10 Da narrow-window scan) versus high-resolution (Agilent 7250 qTOF, GC-Orbitrap full-scan) GC-CI-MS configurations detect isotopologues of central carbon metabolites under isobutane chemical ionization.

## When to use

Use when choosing between or interpreting results from LRMS (QqQ SIM/NS) and HRMS (qTOF, GC-Orbitrap) acquisition for stable-isotope tracing, or when tabulating instrument-specific isotopologue coverage and isotope ratio fidelity.

## When NOT to use

- When the data were acquired with EI (70 eV or 15 eV) instead of CI — extensive molecular-ion fragmentation makes the [M+H]+ envelope comparison invalid
- When comparing against methane-CI spectra — methane induces unwanted [M+H]+ fragmentation, so intensity and coverage comparisons would be confounded
- When the study target is outside GC-amenable central carbon metabolites (e.g. un-derivatized LC-MS analytes) — GC separation efficiency is the stated basis of the advantage

## Inputs

- GC-CI-MS raw chromatograms acquired on an Agilent 7890A GC / 7000 QqQ in SIM or narrow-window (~10 Da) scan mode
- GC-CI-MS full-scan acquisitions on Agilent 7890B GC / 7250 qTOF and GC-Orbitrap instruments
- U-13C-glucose or U-13C-arginine labeled and non-labeled cell extract chromatograms (methoxyamination + MSTFA/1% TMCS derivatized)
- Reported performance statements (isotopologue coverage, [M+H]+ intensity, isotope ratio fidelity) from the article's results section

## Outputs

- Comparison table (exportable as CSV) of configurations vs. reagent gas, mass resolution, isotopologue coverage of central carbon metabolites, and performance notes
- isoSCAN-quantified isotopologue distributions per metabolite for each LR/HR configuration
- An instrument-selection recommendation (HRMS full scan vs. QqQ SIM) grounded in sensitivity and isotope ratio fidelity tradeoffs

## How to apply

Build a configuration-by-metric comparison table with rows for QqQ SIM, QqQ narrow-window (~10 Da) scan, GC-qTOF full scan, and GC-Orbitrap full scan, all run with isobutane as CI reagent gas, and columns for reagent gas, mass resolution, isotopologue coverage of glycolytic/TCA intermediates, and qualitative performance notes. Populate values only from article-reported evidence: isobutane concentrates abundance in the [M+H]+ isotopic envelope and gives a >5-fold more intense [M+H]+ ion than methane for glycolytic and TCA intermediates, while EI at 70 eV (and even 15 eV) fragments molecular ions and is less suitable for enrichment detection. Use isoSCAN to process chromatograms from both LRMS and HRMS so isotopologue quantification is comparable across instruments. Apply the key interpretive finding: latest-generation HR full-scan instruments reach similar sensitivity to QqQ-based analyzers, but isotope ratio fidelity is superior in LRMS — so select HRMS for isotopologue coverage and LRMS-SIM for quantitative ratio precision.

## Related tools

- **isoSCAN** (R package that automatically quantifies all isotopologues from GC-CI-MS data of both LRMS and HRMS labeling studies, enabling cross-instrument comparison) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment for installing and running isoSCAN on the QqQ and HR full-scan datasets) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
install.packages("devtools", dependencies=TRUE); library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- Each table row cites only article-reported values; the isobutane row shows a >5-fold [M+H]+ intensity advantage over methane for glycolytic pathway and TCA cycle intermediates
- All four configurations (QqQ SIM, QqQ NS ~10 Da windows, GC-qTOF, GC-Orbitrap) are represented with reagent gas, resolution, and coverage fields populated
- The interpretation distinguishes sensitivity equivalence (HR full-scan ~ QqQ sensitivity) from isotope ratio fidelity (superior in LRMS) rather than conflating them
- Isotopologue quantification is produced by isoSCAN from both LRMS and HRMS inputs with the same processing, ensuring a like-for-like comparison
- The comparison is restricted to isobutane-CI acquisitions; any EI-derived rows are flagged as fragmenting and unsuitable for enrichment detection

## Limitations

- HRMS full scan matches QqQ sensitivity but loses isotope ratio fidelity, so it is not interchangeable with LRMS-SIM for precise ratio measurements
- Methane CI fragments the [M+H]+ ion, limiting any comparison made with methane as reagent gas
- Cell culturing protocols are only referenced to previously published work rather than fully described, so reproduction depends on external sources
- No software version or changelog is provided for isoSCAN in the methods, so exact package version at time of analysis is unrecorded

## Evidence

- [abstract] Isobutane as reagent gas in combination with both low-resolution (LR) and high-resolution (HR) MS showed the best performance, enabling precise detection of isotopologues: [evidence span withheld — anchor status: not_found]
- [abstract] triple quadrupole (QqQ) MS in SIM mode, or selected ion clusters using optimized narrow-windows (~10 Da) in scan mode: [evidence span withheld — anchor status: not_found]
- [methods] A 7890A GC system coupled to a 7000 QqQ mass spectrometer (Agilent Technologies) was used for isotopologue determination: [evidence span withheld — anchor status: not_found]
- [results] where the [M+H]+ ion was, on average, >5-fold more intense using isobutane than methane (Figure 1C): [evidence span withheld — anchor status: not_found]
- [results] we found that CI with isobutane yielded most suitable spectra to detect isotopic enrichments in comparison with EI (Figure 1A): [evidence span withheld — anchor status: not_found]
- [readme] R Package to automatically process GC-CI-MS data for stable isotope labelling experiments: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
