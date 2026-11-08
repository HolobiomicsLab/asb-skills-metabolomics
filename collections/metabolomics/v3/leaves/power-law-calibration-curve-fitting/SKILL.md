---
name: power-law-calibration-curve-fitting
description: Use when you have TWIM-MS arrival times from calibrant mixtures (e.g.,
  Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix) plus reference
  DTCCSN2 values, and need accurate CCS calibration of unknown features without prior
  identification.
when_to_use_negative:
- Data acquired in negative ionization mode — MOCCal is not yet extended to negative
  ionization mode data sets
- Chemical classes outside small molecules, lipids, and peptides (e.g., other endogenous
  or xenobiotic classes) on TWIM-MS platforms
- Drift-tube or trapped IM instruments — the power law formulation and arrival-time
  terminology are specific to traveling wave IM-MS
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python application that fits the class-specific power law calibrations, assigns
    unknown features to the best calibration by smallest ΔCCS, and outputs calibrated
    TWCCSN2 values
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used to extract arrival time data for the calibrants from raw .mzML files
    and fit Gaussian distributions whose apexes are recorded as arrival times (v1.3.2
    required for MOCCal RawDT)
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Traveling wave ion mobility mass spectrometer on which the calibration and
    experimental data were collected
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:f93b980734ae49f4552d4b93ec07072f892a7393102e2e464ddb50c9817dbf4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# power-law-calibration-curve-fitting

## Summary

Fits class-specific power law calibration curves (small molecule, lipid, peptide) relating TWIM arrival times to drift-tube DTCCSN2 reference values, as implemented in MOCCal, so that each unknown feature on a Waters SYNAPT XS-type TWIM-MS platform can be calibrated with the structurally matched equation rather than a single global curve.

## When to use

Use when you have TWIM-MS arrival times from calibrant mixtures (e.g., Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix) plus reference DTCCSN2 values, and need accurate CCS calibration of unknown features without prior identification.

## When NOT to use

- Data acquired in negative ionization mode — MOCCal is not yet extended to negative ionization mode data sets
- Chemical classes outside small molecules, lipids, and peptides (e.g., other endogenous or xenobiotic classes) on TWIM-MS platforms
- Drift-tube or trapped IM instruments — the power law formulation and arrival-time terminology are specific to traveling wave IM-MS

## Inputs

- Raw calibration data as .mzML files (e.g., Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix, SPLASH Lipidomix, MassPREP enolase digest)
- Reference DTCCSN2 values and DTCCSN2 vs m/z trendlines from the Unified CCS Compendium
- Experimental feature spreadsheet containing feature identifiers, m/z, charge state, retention time, and arrival time values
- Processed calibration data (arrival time + DTCCSN2 pairs) when using the UserDT executable variant

## Outputs

- Class-specific power law calibration parameters (small molecule, lipid, peptide regressions)
- Per-feature calibrated TWCCSN2 values from each candidate class-specific curve
- Per-feature class assignment with ΔCCS (Å2) against the assigned class trendline and an assignment score (0–4)
- feature_class_assignments.csv class-assignment table

## How to apply

Extract calibrant arrival times from .mzML raw data using DEIMoS (v1.3.2), fitting each feature to a Gaussian distribution and recording the apex as the arrival time. Optimize separate power law regression parameters for each biomolecular class (small molecule, lipid, peptide) using the calibrant's arrival time and its reference drift tube CCS (DTCCSN2) value from the Unified CCS Compendium, since each class has distinct fit parameters. Import experimental features as a spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time, and calibrate each singly charged feature's arrival time with up to three class-specific equations to obtain candidate TWCCSN2 values. For class assignment, compute ΔCCS as the absolute difference (in Å2) between each candidate TWCCSN2 value and the class's DTCCSN2 vs m/z regression line, enforce m/z constraints (small molecules < m/z 550, lipids > m/z 300, falling back to the next closest match by ΔCCS if violated), and select the calibration whose curve yields the smallest ΔCCS. Record the calibrated TWCCSN2 value, ΔCCS, and a calibration effect score (1–4) per feature; score 4 means ≥6% difference from the next closest match, score 3 = 3–6%, score 2 = 1–3%, score 1 = <1%, and multiply charged assignments default to 0.

## Related tools

- **MOCCal** (Python application that fits the class-specific power law calibrations, assigns unknown features to the best calibration by smallest ΔCCS, and outputs calibrated TWCCSN2 values) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used to extract arrival time data for the calibrants from raw .mzML files and fit Gaussian distributions whose apexes are recorded as arrival times (v1.3.2 required for MOCCal RawDT)) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Traveling wave ion mobility mass spectrometer on which the calibration and experimental data were collected)

## Examples

```
conda activate deimos && python MOCCal_RawDT.py  # run MOCCal raw-data variant inside the DEIMoS 1.3.2 virtual environment; alternatively use the prebuilt MOCCal executable with an 'Output' folder present
```

## Evaluation signals

- Each row of feature_class_assignments.csv contains a class label, a calibrated TWCCSN2 value, and a ΔCCS computed against the assigned class's DTCCSN2 vs m/z trendline
- Assignment accuracy for lipids and singly charged peptides falls within the reported MOCCal envelope: ≥70% correct assignment with calibration errors of ±1% (best results at ±0.5%)
- Small molecule features from the CCS Compendium are assigned to the small molecule calibration 81–83% of the time with errors within ±0.5 to 5.0%
- Sanity check against the published example: a lipid feature at m/z 692.517 calibrated with the lipid power law should sit ~0.8 Å2 (0.18% error) from the lipid trendline, versus 38.7 Å2 (small molecule) and 25.3 Å2 (peptide) for the wrong curves
- Assignment scores are within the defined range: score 4 = ≥6% difference from next closest match, 3 = 3–6%, 2 = 1–3%, 1 = <1%, multiply charged = 0

## Limitations

- MOCCal is currently scoped around small molecule, lipid, and peptide calibrations within TWIM-MS constraints
- Not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes (user-driven expansion welcomed)
- Applying a single calibration equation across mixed multiomic features introduces calibration biases that may confound feature identification — hence the need for class-specific curves
- TWIM platforms record detector arrival time, not true drift time, so the terms are used interchangeably only as a software convenience
- If a feature falls outside the m/z constraints for its ΔCCS-assigned class, the next closest match by ΔCCS is selected, which may degrade accuracy

## Evidence

- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] Parameters are optimized for class-specific power law regressions using the calibrant's arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [results] arrival time data for the calibrants are extracted using the open-source software DEIMoS and fit to a Gaussian distribution.: [evidence span withheld — anchor status: not_found]
- [results] The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] less than m/z 550 for small molecules; greater than m/z 300 for lipids. If the feature falls outside the m/z constraints for the assigned class, the next closest match by ΔCCS will be selected: [evidence span withheld — anchor status: not_found]
- [discussion] User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed: "User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed"
- [readme] If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos).: "If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)."
