---
name: central-carbon-metabolite-isotopologue-annotation
description: Use when processing GC-CI-MS isotope-labelling data (e.g. U-13C-glucose
  or U-13C-arginine cell cultures) to quantify isotopologue distributions of glycolytic
  and TCA cycle metabolites across LRMS and HRMS acquisitions.
when_to_use_negative:
- When the goal is peak deconvolution of complex EI spectra rather than quantifying
  the intact [M+H]+ isotopic envelope — EI at 70 eV causes extensive fragmentation
  that this skill does not address.
- When the analytes are outside central carbon metabolism (e.g. lipids or specialized
  metabolites) whose chromatography or ionization behaviour differs from the glycolytic/TCA
  panel described here.
- When input is already a processed isotopologue table and the task is downstream
  statistical analysis rather than raw-data isotopologue annotation.
edam_operation: http://edamontology.org/operation_3765
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0621
tools:
- name: isoSCAN
  role: R package that automatically quantifies all isotopologues from GC-CI-MS data
    in LRMS and HRMS stable-isotope labelling studies
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment for executing isoSCAN isotopologue quantification
- name: Agilent 7890A GC / 7000 QqQ mass spectrometer
  role: LRMS acquisition in SIM and narrow-window scan (~10 Da) modes for isotopologue
    determination
- name: Agilent 7890B GC / 7250 QTOF mass spectrometer
  role: HRMS full-scan acquisition used to compare isotope ratio fidelity against
    LRMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:7ecc7cb71e4b304ec73cf4a70424c7f20b02f7fb3e6ea2a35318288ce09ff83d
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
  edam_operation: http://edamontology.org/operation_3765
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# central-carbon-metabolite-isotopologue-annotation

## Summary

Annotate and quantify isotopologues of central carbon metabolites (glycolysis and TCA cycle intermediates) from GC-CI-MS data acquired with isobutane reagent gas on low-resolution (SIM/narrow-window scan) and high-resolution (qTOF, GC-Orbitrap) instruments, using isoSCAN to process LRMS and HRMS raw data into isotopologue tables.

## When to use

Use when processing GC-CI-MS isotope-labelling data (e.g. U-13C-glucose or U-13C-arginine cell cultures) to quantify isotopologue distributions of glycolytic and TCA cycle metabolites across LRMS and HRMS acquisitions.

## When NOT to use

- When the goal is peak deconvolution of complex EI spectra rather than quantifying the intact [M+H]+ isotopic envelope — EI at 70 eV causes extensive fragmentation that this skill does not address.
- When the analytes are outside central carbon metabolism (e.g. lipids or specialized metabolites) whose chromatography or ionization behaviour differs from the glycolytic/TCA panel described here.
- When input is already a processed isotopologue table and the task is downstream statistical analysis rather than raw-data isotopologue annotation.

## Inputs

- GC-CI-MS raw data files from derivatized cell extracts (methoxyamine + MSTFA/TMCS) acquired with isobutane reagent gas
- Isotope-labelling cell culture conditions (e.g. U-13C-glucose or U-13C-arginine replaced medium)
- Instrument configuration metadata: 7890A GC / 7000 QqQ (SIM or narrow-window scan) or 7890B GC / 7250 QTOF / GC-Orbitrap full scan

## Outputs

- Automatically quantified isotopologue distribution table per metabolite (via isoSCAN)
- CSV export comparing tested configurations (LRMS SIM / NS, qTOF, GC-Orbitrap) with isotopologue coverage and performance notes
- [M+H]+ ion intensity comparison demonstrating >5-fold advantage of isobutane over methane

## How to apply

Start with derivatized cell extracts prepared by methoxyamination (methoxyamine in pyridine, 45 min at 60 °C) followed by silylation (MSTFA + 1% TMCS, 30 min at 60 °C), then acquire isotopologue data by GC coupled to chemical ionization with isobutane reagent gas. Configure the instrument according to the metabolite panel: a 7890A GC / 7000 QqQ in SIM or narrow-window scan (~10 Da windows) mode for LRMS, or a 7890B GC / 7250 QTOF / GC-Orbitrap in full-scan mode for HRMS. The oven program for TCA-cycle metabolites sits at 70 °C for 1 min and ramps at 10 °C/min to 325 °C; polyamine gradients use 5 °C/min (70→150 °C), 10 °C/min (150→250 °C), and 50 °C/min (250→325 °C). Rationale for choosing isobutane: it concentrates the metabolite signal in the intact [M+H]+ ion — on average >5-fold more intense than methane for glycolytic and TCA intermediates — and outperforms EI at 70 eV and low-energy EI at 15 eV for isotopic enrichment detection. Process raw data with the isoSCAN R package (install via devtools::install_github("jcapelladesto/isoSCAN")) to automatically quantify all isotopologues and export a CSV summary. Judge correctness by confirming that the isotope ratio fidelity in LRMS matches or exceeds HRMS (a reported finding), and that the isotopologue coverage of central carbon metabolites is consistent across configurations.

## Related tools

- **isoSCAN** (R package that automatically quantifies all isotopologues from GC-CI-MS data in LRMS and HRMS stable-isotope labelling studies) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment for executing isoSCAN isotopologue quantification)
- **Agilent 7890A GC / 7000 QqQ mass spectrometer** (LRMS acquisition in SIM and narrow-window scan (~10 Da) modes for isotopologue determination)
- **Agilent 7890B GC / 7250 QTOF mass spectrometer** (HRMS full-scan acquisition used to compare isotope ratio fidelity against LRMS)

## Examples

```
install.packages("devtools", dependencies=TRUE); library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- Verify that the [M+H]+ ion intensity using isobutane is on average >5-fold greater than with methane for glycolytic and TCA cycle intermediates.
- Check that isoSCAN automatically quantifies all isotopologues of the targeted central carbon metabolites without manual peak picking.
- Confirm the isotope ratio fidelity observed in LRMS is comparable to (or better than) HRMS, as reported in the article.
- Validate the exported CSV has one row per tested configuration (LRMS SIM, LRMS NS, qTOF, GC-Orbitrap) with populated isotopologue coverage columns.
- Ensure derivatization steps (methoxyamination at 60 °C for 45 min, silylation at 60 °C for 30 min) are recorded and reproducible before GC-CI-MS acquisition.

## Limitations

- IsoSCAN is designed for GC-CI-MS data with isobutane reagent gas; performance with other reagent gases or ionization modes is not established.
- The article does not provide an isoSCAN software version or a standalone code repository URL beyond the GitHub link; users should verify the package vignette for current parameters.
- Cell culturing protocols are referenced to previously published work, so the skill does not prescribe cell-culture conditions beyond the isotope-labeling substitution described.
- The GC oven temperature programs differ between metabolite panels (TCA cycle vs polyamines); mixing analytes across panels requires careful scheduling.

## Evidence

- [abstract] isobutane: [evidence span withheld — anchor status: not_found]
- [results] the [M+H]+ ion was, on average, >5-fold more intense using isobutane than methane: [evidence span withheld — anchor status: not_found]
- [abstract] automatic quantification of isotopologues with isoSCAN: "we developed an R-based tool called isoSCAN that automatically quantifies all isotopologues"
- [readme] install_github instruction from README: "install_github("jcapelladesto/isoSCAN")"
- [abstract] narrow-window scan ~10 Da: [evidence span withheld — anchor status: not_found]
