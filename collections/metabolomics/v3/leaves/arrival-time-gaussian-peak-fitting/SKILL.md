---
name: arrival-time-gaussian-peak-fitting
description: Use when raw TWIM-MS calibration or experimental files (.mzML) must be
  converted into feature-level arrival times before power-law CCS calibration, e.g.
  for MOCCal RawDT workflows.
when_to_use_negative:
- You already have a processed feature table with arrival/drift times — use MOCCal's
  UserDT executable path instead, which requires no DEIMoS installation
- Data were acquired on a drift-tube IM-MS platform where measured drift time can
  be used directly rather than extracted from traveling-wave arrival-time distributions
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Open-source tool used to extract arrival-time distributions from raw .mzML
    files and fit them to Gaussians; must be version 1.3.2 and installed in its own
    virtual environment for MOCCal RawDT
  repo: http://github.com/pnnl/deimos
- name: MOCCal
  role: Consumes the Gaussian-fitted apex arrival times together with raw data to
    optimize class-specific power law CCS calibration parameters
  repo: https://github.com/HinesLab/MOCCal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:18b3a43cb0eb900cc75cda692061d2ec9d43e20e7c46b39f713d8bfbcc966853
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCSBase
  - DEIMoS
  - MOCCal
  - Waters AQUITY UPLC BEH amide
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# arrival-time-gaussian-peak-fitting

## Summary

Extracts arrival-time distributions from traveling-wave IM-MS raw data (.mzML) using DEIMoS and fits each feature's arrival-time distribution to a Gaussian, recording the apex as the feature's arrival time for downstream CCS calibration.

## When to use

Use when raw TWIM-MS calibration or experimental files (.mzML) must be converted into feature-level arrival times before power-law CCS calibration, e.g. for MOCCal RawDT workflows.

## When NOT to use

- You already have a processed feature table with arrival/drift times — use MOCCal's UserDT executable path instead, which requires no DEIMoS installation
- Data were acquired on a drift-tube IM-MS platform where measured drift time can be used directly rather than extracted from traveling-wave arrival-time distributions

## Inputs

- Raw TWIM-MS calibration data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, SPLASH Lipidomix, Agilent ESI-L Tune Mix)
- DEIMoS 1.3.2 virtual environment (required for the RawDT MOCCal workflow)

## Outputs

- Per-calibrant (or per-feature) apex arrival times from Gaussian fits of arrival-time distributions
- Arrival-time values formatted into MOCCal data templates for power law calibration

## How to apply

Load raw calibration data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit and SPLASH Lipidomix calibrants) into DEIMoS 1.3.2. For each calibrant ion, extract the arrival-time distribution in the mobility dimension and fit it to a Gaussian distribution; record the apex of the Gaussian fit as the feature's arrival time. MOCCal uses 'arrival time' and 'drift time' interchangeably, but note that TWIM platforms record detector arrival time, not true drift time. The resulting apex arrival times, paired with reference DTCCSN2 values from the Unified CCS Compendium, are the required inputs for fitting class-specific power law regressions (small molecules, lipids, peptides).

## Related tools

- **DEIMoS** (Open-source tool used to extract arrival-time distributions from raw .mzML files and fit them to Gaussians; must be version 1.3.2 and installed in its own virtual environment for MOCCal RawDT) — http://github.com/pnnl/deimos
- **MOCCal** (Consumes the Gaussian-fitted apex arrival times together with raw data to optimize class-specific power law CCS calibration parameters) — https://github.com/HinesLab/MOCCal

## Examples

```
python MOCCal_RawDT.py  # run inside the DEIMoS 1.3.2 virtual environment after loading .mzML calibration files
```

## Evaluation signals

- Each calibrant's arrival-time distribution yields a converged Gaussian fit whose apex is the recorded arrival time (no failed/ambiguous fits for calibrants)
- Fitted arrival times paired with Unified CCS Compendium DTCCSN2 values produce class-specific power law regressions with reasonable fit parameters (distinct parameters per biomolecular class)
- Running MOCCal_RawDT.py inside the DEIMoS 1.3.2 virtual environment completes and writes outputs to the required 'Output' folder
- Downstream MOCCal assignment on NIST SRM 1950 features reproduces expected calibration accuracy (e.g. ~0.28 Å2 lipid RMSE), indicating upstream apex extraction was correct

## Limitations

- DEIMoS version 1.3.2 specifically is required for the RawDT workflow (the DEIMoS README cites version 1.6.2 for its own citation); other versions may not be compatible
- TWIM arrival time is not true drift time — the apex Gaussian value is a detector arrival time, so calibrated CCS values depend on the power law calibration rather than direct measurement
- MOCCal is currently limited to small molecule, lipid, and peptide calibrations and has not been extended to negative ionization mode datasets or other chemical classes

## Evidence

- [results] arrival time data for the calibrants are extracted using the open-source software DEIMoS and fit to a Gaussian distribution. The apex of the Gaussian distribution is recorded as the feature's arrival: [evidence span withheld — anchor status: not_found]
- [readme] If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos).: "If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)."
- [results] raw data for each calibration solution should be loaded as a .mzML file: "raw data for each calibration solution should be loaded as a .mzML file"
- [readme] TWIM platforms record the time at which the ion reaches the detector (arrival time) rather than the time an ion spends within the TWIM cell (drift time).: "TWIM platforms record the time at which the ion reaches the detector (arrival time) rather than the time an ion spends within the TWIM cell (drift time)."
