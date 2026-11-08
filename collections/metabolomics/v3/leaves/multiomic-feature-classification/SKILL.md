---
name: multiomic-feature-classification
description: Use when you have singly charged feature arrival times from TWIM-MS (e.g.,
  Waters SYNAPT XS) with computed class-specific TWCCSN2 calibration parameters, and
  you must assign each unknown feature to a biomolecular class without prior identification.
when_to_use_negative:
- Features are multiply charged — these receive a default score of 0 and are outside
  the core ΔCCS assignment logic calibrated for singly charged species
- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative
  ionization mode data sets
- Features belong to chemical classes outside small molecules, lipids, and peptides
  (e.g., other endogenous or xenobiotic classes) — MOCCal is currently scoped to these
  three calibrations
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: 'Performs the class assignment: calibrates each arrival time with up to three
    class-specific power law curves, applies m/z constraints, selects the smallest-ΔCCS
    class, and writes the per-feature assignment table'
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: 'Upstream feature extraction from .mzML raw data: extracts arrival times and
    fits them to a Gaussian distribution whose apex is recorded as the feature''s
    arrival time (required when running MOCCal_RawDT.py, version 1.3.2)'
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:36a7d9e1e5ce0332341240d5c9af9920963df8a2d0ff99fc3a57824248c4b74c
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# multiomic-feature-classification

## Summary

MOCCal's biomolecular class assignment step assigns unidentifiable TWIM-MS features to small molecule, lipid, or peptide classes by calibrating each arrival time with up to three class-specific power law curves and selecting the class whose ΔCCS to the DTCCSN2 vs m/z trendline is smallest.

## When to use

Use when you have singly charged feature arrival times from TWIM-MS (e.g., Waters SYNAPT XS) with computed class-specific TWCCSN2 calibration parameters, and you must assign each unknown feature to a biomolecular class without prior identification.

## When NOT to use

- Features are multiply charged — these receive a default score of 0 and are outside the core ΔCCS assignment logic calibrated for singly charged species
- Data were acquired in negative ionization mode — MOCCal is not yet extended to negative ionization mode data sets
- Features belong to chemical classes outside small molecules, lipids, and peptides (e.g., other endogenous or xenobiotic classes) — MOCCal is currently scoped to these three calibrations

## Inputs

- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values
- Class-specific power law calibration parameters (small molecule, lipid, peptide z=1) from the calibration step
- DTCCSN2 vs m/z regression lines from the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)

## Outputs

- Per-feature class-assignment table (feature_class_assignments.csv) with class label, calibrated TWCCSN2 value, ΔCCS, and assignment score (1–4) for each feature

## How to apply

Import the experimental feature spreadsheet (feature identifiers, m/z, charge state, retention time, arrival time) and the calibration parameters from the prior power-law fitting step. For each singly charged feature, compute up to three calibrated TWCCSN2 values using the small molecule, lipid, and peptide (z=1) power law calibration equations. For each candidate, compute ΔCCS as the absolute difference in Å2 between the calibrated TWCCSN2 value and the class's DTCCSN2 vs m/z regression line built from the Unified CCS Compendium. Apply class m/z constraints — small molecules < m/z 550, lipids > m/z 300 — and if a feature falls outside the m/z constraints for the ΔCCS-selected class, fall back to the next closest match by ΔCCS. Assign the class with the smallest ΔCCS, recording the calibrated TWCCSN2 value, ΔCCS, and assignment score (1–4; score 4 = ≥6% difference from next closest match, score 1 = <1%; multiply charged features default to score 0). The rationale is that each biomolecular class has distinct power law fit parameters, so the structurally matched calibration yields the smallest deviation from its trendline (e.g., a lipid feature calibrated with the lipid curve sat 0.8 Å2 from the lipid trendline versus 38.7 and 25.3 Å2 for small molecule and peptide curves).

## Related tools

- **MOCCal** (Performs the class assignment: calibrates each arrival time with up to three class-specific power law curves, applies m/z constraints, selects the smallest-ΔCCS class, and writes the per-feature assignment table) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Upstream feature extraction from .mzML raw data: extracts arrival times and fits them to a Gaussian distribution whose apex is recorded as the feature's arrival time (required when running MOCCal_RawDT.py, version 1.3.2)) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal.py --input features.csv --calibrations calibrations.csv --output Output/feature_class_assignments.csv
```

## Evaluation signals

- Every row of the assignment table contains a class label, a calibrated TWCCSN2 value, and a ΔCCS computed against the assigned class's trendline
- Assignment accuracy falls within the reported MOCCal performance envelope: ≥70% correct assignment for lipids and singly charged peptides with calibration errors of ±1% (best at ±0.5%)
- Small molecule compendium features are assigned to the small molecule calibrations 81–83% of the time with errors of ±0.5 to 5.0%
- For the assigned class, the calibrated TWCCSN2 value should be near its DTCCSN2 (e.g., the lipid example was 0.18% error via the lipid curve versus 1.8% and 2.6% via small molecule and peptide curves)
- No assigned feature violates its class m/z constraint (small molecules < m/z 550; lipids > m/z 300) — violations must have been reassigned to the next closest match by ΔCCS

## Limitations

- MOCCal's class assignment is built around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms; other chemical classes are not covered
- MOCCal is not yet extended to negative ionization mode data sets; user-driven expansion is welcomed
- Applying a single (wrong-class) calibration equation introduces calibration biases that may confound feature identification — this is precisely the error the class assignment step mitigates
- Multiply charged features receive a default score of 0, so the 1–4 scoring scale only meaningfully discriminates singly charged assignments

## Evidence

- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [abstract] Selection by ΔCCS against trendlines: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] m/z constraints and fallback rule: [evidence span withheld — anchor status: not_found]
- [results] Performance envelope for correct assignment: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [results] Worked example of class discrimination via ΔCCS: "the feature at m/z 692.517 (a lipid) is 0.8 Å2 from the lipid trendline. The same feature calibrated with the small molecule and peptide (z = 1) power law equations is 38.7 Å2 and 25.3 Å2"
- [results] Input format for experimental features: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
