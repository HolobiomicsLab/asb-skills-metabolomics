---
name: nmr-data-format-parsing
description: Use when raw 1D 1H NMR spectra in Bruker IVDr format (600 MHz) must be
  loaded and made ppm-addressable before spectral calibration to the glucose anomeric
  proton at 5.233 ppm.
when_to_use_negative:
- Input is already a processed, calibrated spectrum or a metabolite concentration
  table — parsing raw Bruker IVDr files would redo completed work
- Data are 2D spectra (e.g., 2D-Jres analyses used only for validating the 22 metabolites'
  assignments) rather than standard one-dimensional 1H NMR spectra
- Spectra were not acquired on Bruker IVDr 600 MHz instrumentation and no Bruker IVDr-format
  files are available to parse
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SMolESY-select
  role: 'Downstream routine that consumes the parsed spectrum: it automatically calibrates
    every spectrum to the 1H NMR signal of the glucose anomeric proton at 5.233 ppm'
  repo: https://github.com/pantakis/SMolESY-select
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument whose acquisition format defines the raw input files that must
    be parsed (IVDr-format 1D 1H NMR at 600 MHz)
- name: SMolESY
  role: Spectral enhancement pipeline that operates on the parsed spectra, providing
    baseline suppression and resolution enhancement before metabolite assignment
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:aa2c5252d580d8c145cef6872fece5e1d43a33a354ae9efebb52c5846443164e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - MTBLS395
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nmr-data-format-parsing

## Summary

Reads raw 1D 1H NMR spectra acquired in Bruker IVDr format on 600 MHz spectrometers (e.g., the MTBLS395 serum/plasma cohort) into a ppm-indexed array ready for downstream SMolESY-select calibration and metabolite assignment.

## When to use

Use when raw 1D 1H NMR spectra in Bruker IVDr format (600 MHz) must be loaded and made ppm-addressable before spectral calibration to the glucose anomeric proton at 5.233 ppm.

## When NOT to use

- Input is already a processed, calibrated spectrum or a metabolite concentration table — parsing raw Bruker IVDr files would redo completed work
- Data are 2D spectra (e.g., 2D-Jres analyses used only for validating the 22 metabolites' assignments) rather than standard one-dimensional 1H NMR spectra
- Spectra were not acquired on Bruker IVDr 600 MHz instrumentation and no Bruker IVDr-format files are available to parse

## Inputs

- Raw 1D 1H NMR spectrum in Bruker IVDr format (600 MHz) from serum/plasma samples, e.g. MTBLS395 from the Metabolights repository (https://www.ebi.ac.uk/metabolights)

## Outputs

- Parsed ppm-indexed 1D 1H NMR spectral array (all data points preserved) suitable for glucose-based calibration at 5.233 ppm
- Recorded ppm shift offset applied during subsequent calibration

## How to apply

Load the raw 1D 1H NMR spectrum (600 MHz, Bruker IVDr format) acquired from a serum or plasma sample, such as those in the MTBLS395 Metabolights cohort. Verify the acquisition instrument is a Bruker IVDr 600 MHz spectrometer so that chemical-shift positions are interpretable on the expected ppm axis. Once loaded, the parsed spectrum is handed to the SMolESY-select calibration routine, which locates the glucose anomeric proton resonance and computes the ppm offset needed to bring it to 5.233 ppm. The parsing step must preserve all data points of the spectrum so the computed shift can be applied uniformly across the full spectral vector. Record the ppm shift offset applied together with the calibrated output spectrum so downstream metabolite-panel integration steps remain traceable.

## Related tools

- **SMolESY-select** (Downstream routine that consumes the parsed spectrum: it automatically calibrates every spectrum to the 1H NMR signal of the glucose anomeric proton at 5.233 ppm) — https://github.com/pantakis/SMolESY-select
- **Bruker IVDr 600 MHz spectrometer** (Instrument whose acquisition format defines the raw input files that must be parsed (IVDr-format 1D 1H NMR at 600 MHz))
- **SMolESY** (Spectral enhancement pipeline that operates on the parsed spectra, providing baseline suppression and resolution enhancement before metabolite assignment) — https://github.com/pantakis/SMolESY

## Evaluation signals

- After the calibration step that consumes the parsed spectrum, the glucose anomeric proton signal aligns to 5.233 ppm
- The parsed spectral vector retains all original data points so the computed ppm shift can be applied uniformly to the whole spectrum
- The ppm shift offset applied is recorded and written out alongside the calibrated spectrum
- The parsed spectrum's ppm axis is consistent with 600 MHz Bruker IVDr acquisition (e.g., the expected glucose anomeric region ~5.233 ppm is locatable)

## Limitations

- No software version is stated for SMolESY-select or SMolESY, so parsed-file compatibility across tool versions is unverified
- The >8800-sample validation dataset of serum/plasma spectra is not named or given an accession/repository, limiting reproduction of the parsing-to-assignment pipeline beyond MTBLS395
- Operation is supported by a GUI (Figure S9, Video S1) available only as supplementary artifacts, which are not included in the article text

## Evidence

- [methods] IVDr format input for parsing: "NMR samples were acquired using Bruker IVDr 600 MHz spectrometers"
- [results] MTBLS395 serum cohort source data: "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [other] Raw-file loading step feeding calibration: [evidence span withheld — anchor status: not_found]
- [results] Calibration target after parsing: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [results] Calibrated glucose signal position: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
