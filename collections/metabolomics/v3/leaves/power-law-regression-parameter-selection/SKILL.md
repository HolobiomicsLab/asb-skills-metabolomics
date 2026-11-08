---
name: power-law-regression-parameter-selection
description: Use when processing Waters SYNAPT XS TWIM calibration data — e.g. Major
  Mix, SPLASH Lipidomix, or Agilent Tune Mix — to derive class-specific calibration
  parameters for later CCS assignment.
when_to_use_negative:
- Experimental (unknown) feature data rather than calibrant data — parameter fitting
  requires reference DTCCSN2 values known for each calibrant
- Drift-tube or TIMS CCS instruments, where direct measured drift time replaces traveling-wave
  power law calibration
- Negative ionization mode datasets or chemical classes beyond small molecule/lipid/peptide
  — the article notes MOCCal is not yet extended to these
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Executes the power law regression and parameter optimization over class-specific
    calibrant tables
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts calibrant arrival times from raw .mzML via Gaussian fitting (RawDT
    mode prerequisite)
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: TWIM-MS instrument that records the arrival times being calibrated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:f87d77888ad62ed13724c05f33ccf6e0869d2450c1ffd2418750cf0eebc71cf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# power-law-regression-parameter-selection

## Summary

Fit class-specific power law CCS calibration curves (small molecule, lipid, peptide) by regressing calibrant arrival time against reference DTCCSN2 values, yielding the parameters used by MOCCal to convert TWIM arrival times into calibrated TWCCSN2 values.

## When to use

Use when processing Waters SYNAPT XS TWIM calibration data — e.g. Major Mix, SPLASH Lipidomix, or Agilent Tune Mix — to derive class-specific calibration parameters for later CCS assignment.

## When NOT to use

- Experimental (unknown) feature data rather than calibrant data — parameter fitting requires reference DTCCSN2 values known for each calibrant
- Drift-tube or TIMS CCS instruments, where direct measured drift time replaces traveling-wave power law calibration
- Negative ionization mode datasets or chemical classes beyond small molecule/lipid/peptide — the article notes MOCCal is not yet extended to these

## Inputs

- Raw calibration .mzML files (Waters SYNAPT XS TWIM-MS) for RawDT mode
- Pre-processed spreadsheet of calibrant feature identifiers with m/z, charge state, retention time, arrival time, and reference DTCCSN2 values (UserDT mode)
- Calibrant mixtures: Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, Agilent ESI-L Tune Mix, MassPREP enolase digest, poly-dl-alanine

## Outputs

- Optimized class-specific power law calibration parameter sets (small molecule, lipid, peptide) for each charge state
- Calibrated arrival-time-to-CCS regression coefficients consumed by MOCCal's downstream class-assignment and TWCCSN2 calculation steps

## How to apply

Load raw calibration .mzML files and extract each calibrant's arrival time with DEIMoS, fitting the drift-time distribution to a Gaussian and recording the apex. Assemble a table pairing each calibrant's arrival time with its literature DTCCSN2 value and charge state, then run MOCCal's parameter optimization to fit separate power law regressions (Ω' = A·t'ᴬ^B·z'ᶜ style, per class: small molecule, lipid, peptide z=1 and z≥2). Because each biomolecular class has distinct fit parameters, do not pool classes into one regression; MOCCal optimizes parameters for each class independently. For processed (UserDT) inputs, skip extraction and supply a spreadsheet of pre-measured arrival times and reference CCS values directly. The resulting parameter sets are then applied to unknown experimental features, each feature being calibrated with up to three class-specific equations before selection by the ΔCCS regression-line criterion.

## Related tools

- **MOCCal** (Executes the power law regression and parameter optimization over class-specific calibrant tables) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts calibrant arrival times from raw .mzML via Gaussian fitting (RawDT mode prerequisite)) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (TWIM-MS instrument that records the arrival times being calibrated)

## Examples

```
python MOCCal_RawDT.py  # in DEIMoS v1.3.2 environment, per README RawDT instructions
```

## Evaluation signals

- Fitted power law reproduces calibrant DTCCSN2 within ~±1% for the correct class (article reports lipid calibration at 0.18% error and lipids/peptides at ≥70% correct assignment with ±0.5% errors)
- Each biomolecular class yields its own distinct fit parameters — pooled single-class fits are a red flag
- Arrival times extracted from .mzML must match the Gaussian apex, not the mean or median of the drift-time distribution
- Downstream ΔCCS selection must show the true class's curve closest to its trendline (e.g. the m/z 692.517 lipid lands 0.8 Å² from the lipid trendline vs 38.7 Å²/25.3 Å² for small molecule/peptide fits)
- Parameters are charge-state aware: separate regressions for z=1 and z≥2 peptides

## Limitations

- Calibration biases introduced by a single pooled power law can confound feature identification — class-specific fits exist precisely to avoid this
- Terms arrival time and drift time are used interchangeably in the software, but TWIM records detector arrival time, not true drift time
- Only positive ionization and small molecule/lipid/peptide classes are currently supported; user-driven expansion is welcomed

## Evidence

- [results] Distinct class fit parameters: "each biomolecular class has distinct fit parameters for the power law calibration curve"
- [results] Gaussian apex extraction: [evidence span withheld — anchor status: not_found]
- [results] Parameter optimization: [evidence span withheld — anchor status: not_found]
- [abstract] MOCCal automates calibration selection: "Here we describe the development of a multiomic CCS calibration tool, MOCCal, to automate the assignment of unknown features to the power law calibration that provides the most accurate CCS value."
- [readme] MOCCal available at GitHub: "MOCCal, or Multi-Omic CCS Calibrator, is a Python application for high-dimensional, multi-omic traveling-wave ion mobility mass spectrometry (TWIM-MS) data anlaysis."
