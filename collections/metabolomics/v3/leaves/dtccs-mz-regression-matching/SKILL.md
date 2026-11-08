---
name: dtccs-mz-regression-matching
description: Use when processing singly charged TWIM-MS features (m/z, charge state,
  retention time, arrival time) from a Waters SYNAPT XS-type platform and you must
  assign each feature to small molecule, lipid, or peptide class before reporting
  a calibrated TWCCSN2 value.
when_to_use_negative:
- Features are multiply charged — MOCCal assigns them a default score of 0 and the
  three-class ΔCCS competition is designed for singly charged ions
- The analyte class is outside the small molecule/lipid/peptide scope (e.g., other
  endogenous or xenobiotic chemical classes) or data were acquired in negative ionization
  mode, which MOCCal does not yet support
- You already have confident structural identifications for all features and can apply
  a structurally matched calibration directly without class assignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Executes the class-specific power law calibration of arrival times, computes
    ΔCCS against the DTCCSN2 vs m/z trendlines, and assigns each feature the class
    with the smallest ΔCCS
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival time distributions from raw mzML calibration/experimental
    data and fits them to a Gaussian to record feature arrival times prior to class
    assignment
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:3ff366f92d6a329c2ff24e107c83260501d983e346394bfe6dbb57b8e91a039d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dtccs-mz-regression-matching

## Summary

Assigns an unknown TWIM-MS feature to its biomolecular class (small molecule, lipid, or peptide) by calibrating its arrival time with up to three class-specific power law curves and selecting the class whose DTCCSN2 vs m/z trendline from the Unified CCS Compendium yields the smallest ΔCCS.

## When to use

Use when processing singly charged TWIM-MS features (m/z, charge state, retention time, arrival time) from a Waters SYNAPT XS-type platform and you must assign each feature to small molecule, lipid, or peptide class before reporting a calibrated TWCCSN2 value.

## When NOT to use

- Features are multiply charged — MOCCal assigns them a default score of 0 and the three-class ΔCCS competition is designed for singly charged ions
- The analyte class is outside the small molecule/lipid/peptide scope (e.g., other endogenous or xenobiotic chemical classes) or data were acquired in negative ionization mode, which MOCCal does not yet support
- You already have confident structural identifications for all features and can apply a structurally matched calibration directly without class assignment

## Inputs

- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- Class-specific power law calibration parameters (small molecule, lipid, peptide) from the prior calibration step
- DTCCSN2 vs m/z regression lines derived from Unified CCS Compendium values (Supporting Information Document 2, Tables 1–3)

## Outputs

- feature_class_assignments.csv: per-feature table with assigned biomolecular class, calibrated TWCCSN2 value, ΔCCS (Å2), and assignment score (0–4)

## How to apply

For each singly charged feature, compute up to three calibrated TWCCSN2 values using the class-specific power law calibration parameters (small molecule, lipid, peptide z=1) produced in the calibration step. For each candidate, calculate ΔCCS as the absolute difference in Å2 between the calibrated TWCCSN2 value and the class's DTCCSN2 vs m/z regression line built from Unified CCS Compendium values. Enforce m/z constraints — small molecules < m/z 550, lipids > m/z 300 — and if a feature violates the closest match's constraint, fall back to the next closest match by ΔCCS. Assign the class with the smallest ΔCCS and record the calibrated value, ΔCCS, and the calibration effect score (1–4, where 4 means ≥6% difference between the assigned-class CCS and the next closest match, 3 = 3–6%, 2 = 1–3%, 1 = <1%; multiply charged features default to score 0).

## Related tools

- **MOCCal** (Executes the class-specific power law calibration of arrival times, computes ΔCCS against the DTCCSN2 vs m/z trendlines, and assigns each feature the class with the smallest ΔCCS) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival time distributions from raw mzML calibration/experimental data and fits them to a Gaussian to record feature arrival times prior to class assignment) — http://github.com/pnnl/deimos

## Examples

```
MOCCal.exe  # UserDT executable run from a folder containing an 'Output' directory; imports the feature spreadsheet and calibration parameters, writes feature_class_assignments.csv
```

## Evaluation signals

- Every row of feature_class_assignments.csv contains a class label, a calibrated TWCCSN2 value, and a ΔCCS computed against the assigned class's trendline
- Assignment accuracy falls within the reported MOCCal envelope: ≥70% correct assignment for lipids and singly charged peptides with calibration errors of ±1%
- Small molecule features from the CCS Compendium are assigned to small molecule calibrations 81–83% of the time with errors of ±0.5 to 5.0%
- For a correctly class-assigned feature, ΔCCS to the assigned trendline should be far smaller than to competing trendlines (e.g., a lipid at m/z 692.517 is 0.8 Å2 from the lipid trendline but 38.7 and 25.3 Å2 from small molecule and peptide trendlines)
- No assigned feature violates its class m/z constraint (small molecule < m/z 550, lipid > m/z 300); constraint violations trigger the next-closest ΔCCS match instead

## Limitations

- MOCCal's assignment is currently built around small molecule, lipid, and peptide calibrations only, intended for mixed multiomic analyses on TWIM-MS platforms
- MOCCal is not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Applying a single (non-class-matched) power law calibration introduces calibration biases that may confound feature identification
- The RawDT mode requires installing pnnl's DEIMoS, version 1.3.2, in a dedicated virtual environment

## Evidence

- [abstract] ΔCCS selection rule: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] m/z constraints and fallback: [evidence span withheld — anchor status: not_found]
- [results] Example lipid assignment with ΔCCS magnitudes: "the feature at m/z 692.517 (a lipid) is 0.8 Å2 from the lipid trendline. The same feature calibrated with the small molecule and peptide (z = 1) power law equations is 38.7 Å2 and 25.3 Å2"
- [results] Performance envelope: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [results] Experimental input format: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
