---
name: twim-ccs-calibration-parameter-fitting
description: Use when processing traveling-wave ion mobility MS data (e.g. Waters
  SYNAPT XS) and you need calibrated TWCCSN2 values for features whose biomolecular
  class is not yet identified.
when_to_use_negative:
- Input is already identified features with known DTCCSN2 values — no calibration/assignment
  needed
- Data are from non-traveling-wave IM-MS platforms (e.g. drift-tube, TIMS) where arrival-time
  ≠ drift-time and power-law parameters differ
- Calibrants fall outside the m/z constraints for the class (small molecules >550,
  lipids <300) — assignment will fall back to next-closest ΔCCS match
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Executes class-specific power-law calibration parameter optimization, feature
    class assignment, and TWCCSN2 calculation for TWIM-MS data.
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival-time distributions from raw .mzML calibration files and fits
    them to Gaussians to obtain the apex (arrival time) fed into MOCCal's power-law
    fits.
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:665dad4dfe0761cbc7914106be512c94d9e034c61ebd224e73233101d8bb51ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - Agilent ESI-L Tune Mix
  - DEIMoS
  - MOCCal
  - NIST SRM 1950
  - Waters Major Mix IMS/Tof Calibration Kit
  - poly-dl-alanine
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# twim-ccs-calibration-parameter-fitting

## Summary

Fit class-specific (small molecule, lipid, peptide) power-law calibration equations relating experimental TWIM arrival times to reference drift-tube CCS (DTCCSN2) values, using calibrants like poly-dl-alanine, Waters Major Mix, and Agilent ESI-L Tune Mix.

## When to use

Use when processing traveling-wave ion mobility MS data (e.g. Waters SYNAPT XS) and you need calibrated TWCCSN2 values for features whose biomolecular class is not yet identified.

## When NOT to use

- Input is already identified features with known DTCCSN2 values — no calibration/assignment needed
- Data are from non-traveling-wave IM-MS platforms (e.g. drift-tube, TIMS) where arrival-time ≠ drift-time and power-law parameters differ
- Calibrants fall outside the m/z constraints for the class (small molecules >550, lipids <300) — assignment will fall back to next-closest ΔCCS match

## Inputs

- .mzML raw calibration files (Waters SYNAPT XS TWIM-MS)
- Calibrant arrival time data (poly-dl-alanine, Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix)
- Reference DTCCSN2 values from the Unified CCS Compendium
- Experimental feature table with feature identifiers, m/z, charge state, retention time, and arrival time values

## Outputs

- Class-specific power-law calibration parameters (small molecule, lipid, peptide z=1/2/3)
- Calibrated TWCCSN2 values per experimental feature
- Per-class calibration error percentages and RMSE values (e.g. 0.02/0.75/0.01/0.06/0.13% and 0.28/0.56/0.09/0.09/0.24)

## How to apply

Load raw calibration data as .mzML files and extract arrival times with DEIMoS v1.3.2, fitting each calibrant's arrival time distribution to a Gaussian and recording the apex. For each biomolecular class, optimize parameters of a power-law regression using the calibrant's arrival time and its reference DTCCSN2 value from the Unified CCS Compendium, yielding separate equations for small molecules, lipids, and peptides at charge states z=1,2,3. Import experimental features as a spreadsheet with identifiers, m/z, charge, retention time, and arrival time; calibrate each singly charged feature with up to three class-specific equations. Select the calibration whose calibrated TWCCSN2 value lies closest (smallest ΔCCS in Å²) to the corresponding DTCCSN2-vs-m/z trendline, respecting m/z constraints (small molecules <550, lipids >300). Validate per-class calibration errors and RMSE against reported values (0.02/0.75/0.01/0.06/0.13% and 0.28/0.56/0.09/0.09/0.24).

## Related tools

- **MOCCal** (Executes class-specific power-law calibration parameter optimization, feature class assignment, and TWCCSN2 calculation for TWIM-MS data.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival-time distributions from raw .mzML calibration files and fits them to Gaussians to obtain the apex (arrival time) fed into MOCCal's power-law fits.) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run inside the DEIMoS v1.3.2 virtual environment after installing pnnl/deimos
```

## Evaluation signals

- Per-class calibration errors match reported values (0.02% lipids, 0.75% small molecules, 0.01/0.06/0.13% peptides z=1/2/3)
- RMSE values match reported table (0.28, 0.56, 0.09, 0.09, 0.24)
- Lipid feature at m/z 692.517 shows ~0.8 Å2 deviation from lipid trendline, vs 38.7 Å2 and 25.3 Å2 under small-molecule/peptide equations
- ≥70% of lipids and singly charged peptides are correctly assigned when calibration errors are ±0.5%
- Small molecules are assigned to the small molecule calibration 81–83% of the time

## Limitations

- MOCCal currently supports only small molecule, lipid, and peptide calibrations within TWIM-MS constraints; other classes or IM platforms are out of scope
- Not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Terms 'arrival time' and 'drift time' are used interchangeably in the software but TWIM records detector arrival time, not true drift time

## Evidence

- [abstract] calibrate every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] Extract arrival times using DEIMoS and fit to a Gaussian distribution: [evidence span withheld — anchor status: not_found]
- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [readme] If you plan to use processed calibration data, you do not need to install Python or any of the packages required to run the code: "If you plan to use MOCCal with processed calibration data, you do not need to install Python or any of the packages required to run the code. To use this version of MOCCal, simply download the MOCCal"
- [results] m/z constraints for class assignment: [evidence span withheld — anchor status: not_found]
