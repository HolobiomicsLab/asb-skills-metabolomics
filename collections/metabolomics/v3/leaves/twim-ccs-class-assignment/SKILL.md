---
name: twim-ccs-class-assignment
description: Use when processing an experimental TWIM feature spreadsheet (feature
  IDs, m/z, charge state, retention time, arrival time) from a Waters SYNAPT-class
  instrument and uncalibrated features must be assigned to a biomolecular class and
  CCS-calibrated without prior identification.
when_to_use_negative:
- Features are multiply charged — MOCCal applies the class-assignment ΔCCS selection
  to singly charged features; multiply charged assignments receive a default score
  of 0
- Data are in negative ionization mode — MOCCal has not yet been extended to negative
  ionization mode data sets
- The features are already identified, so structurally matched calibrations can be
  applied directly without ΔCCS-based class assignment
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: 'Performs the class assignment: calibrates each arrival time with up to three
    class-specific power law curves and selects the curve with smallest ΔCCS to the
    class trendline'
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival time data from calibration .mzML files and fits Gaussian
    distributions whose apexes supply the arrival times used upstream of assignment
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:fb9684fd91b8825fe7b4f87dbca2b5275edd75056d3399fa314ecfb193af1388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# twim-ccs-class-assignment

## Summary

Assigns unknown TWIM-MS features to a biomolecular class (small molecule, lipid, or singly charged peptide) by calibrating each arrival time with up to three class-specific power law curves and selecting the candidate whose calibrated TWCCSN2 value lies closest (smallest ΔCCS, in Å2) to that class's DTCCSN2 vs m/z regression line from the Unified CCS Compendium.

## When to use

Use when processing an experimental TWIM feature spreadsheet (feature IDs, m/z, charge state, retention time, arrival time) from a Waters SYNAPT-class instrument and uncalibrated features must be assigned to a biomolecular class and CCS-calibrated without prior identification.

## When NOT to use

- Features are multiply charged — MOCCal applies the class-assignment ΔCCS selection to singly charged features; multiply charged assignments receive a default score of 0
- Data are in negative ionization mode — MOCCal has not yet been extended to negative ionization mode data sets
- The features are already identified, so structurally matched calibrations can be applied directly without ΔCCS-based class assignment

## Inputs

- Class-specific TWIM power law calibration parameters (small molecule, lipid, peptide) from the calibration step
- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- DTCCSN2 vs m/z regression lines derived from Unified CCS Compendium values

## Outputs

- feature_class_assignments.csv: per-feature class label, calibrated TWCCSN2 value, ΔCCS (Å2), and assignment score (0–4)

## How to apply

Load the class-specific power law calibration parameters (from the prior calibration task) plus the experimental feature spreadsheet containing feature identifiers, m/z, charge state, retention time, and arrival time. For each singly charged feature, compute up to three calibrated TWCCSN2 values using the small molecule, lipid, and peptide (z = 1) power law equations. For each candidate, calculate ΔCCS as the absolute difference (Å2) between the calibrated TWCCSN2 value and the class's DTCCSN2 vs m/z regression line built from Unified CCS Compendium values. Apply m/z constraints — small molecules < m/z 550, lipids > m/z 300 — and if the closest-ΔCCS class violates its constraint, fall back to the next closest match. Record the assigned class, calibrated TWCCSN2, ΔCCS, and a 1–4 assignment score (4 = ≥6% difference from next closest match, 3 = 3–6%, 2 = 1–3%, 1 = <1%; multiply-charged features default to score 0) per feature.

## Related tools

- **MOCCal** (Performs the class assignment: calibrates each arrival time with up to three class-specific power law curves and selects the curve with smallest ΔCCS to the class trendline) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival time data from calibration .mzML files and fits Gaussian distributions whose apexes supply the arrival times used upstream of assignment) — http://github.com/pnnl/deimos

## Examples

```
MOCCal.exe  # run the UserDT executable from a folder containing the calibration parameters, the experimental feature spreadsheet, and an empty 'Output' folder
```

## Evaluation signals

- Every output row contains a class label, a calibrated TWCCSN2 value, and a ΔCCS computed against the assigned class's trendline
- Assignment accuracy falls within the reported MOCCal envelope: ≥70% correct assignment for lipids and singly charged peptides with calibration errors of ±1% (best at ±0.5%)
- Small molecules from the CCS Compendium should be assigned to small molecule calibrations 81–83% of the time with errors of ±0.5 to 5.0%
- Class m/z constraints hold: assigned small molecules < m/z 550, assigned lipids > m/z 300; violations trigger fallback to next closest ΔCCS match
- Assignment score is in range 1–4 for singly charged features (0 for multiply charged) and correlates with the % difference between the assigned and next closest calibrated CCS

## Limitations

- MOCCal is scoped to small molecule, lipid, and peptide calibrations on TWIM-MS platforms; other chemical classes require user-driven expansion
- Negative ionization mode data sets are not yet supported
- A single (mis-matched) calibration equation introduces calibration biases that may confound feature identification — hence the need for class matching
- If the nearest-ΔCCS class violates its m/z constraint, the fallback may not yield the true class

## Evidence

- [results] MOCCal compares each class-calibrated TWCCSN2 value against the corresponding class's DTCCSN2 vs m/z trendline: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] m/z constraints for class assignment: [evidence span withheld — anchor status: not_found]
- [results] assignment score scale: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match."
- [results] performance envelope: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [results] experimental input format: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [discussion] scope limitation: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
- [readme] no identification needed: "MOCCal offers class assignment and CCS calculations without need for identifying the features first."
