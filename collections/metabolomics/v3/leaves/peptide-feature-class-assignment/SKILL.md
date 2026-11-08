---
name: peptide-feature-class-assignment
description: Use when processing traveling-wave IM-MS feature tables (m/z, charge,
  retention time, arrival time) from multiomic experiments where peptide features
  such as ADH or enolase digests and polyalanine peptides must receive class-matched
  CCS calibration without prior identification.
when_to_use_negative:
- When the feature is a small molecule below m/z 550 — the small-molecule calibration,
  not the peptide one, should be assigned per the article's m/z constraints.
- 'When data are negative ionization mode: MOCCal is not yet extended to negative
  mode data sets.'
- When the goal is drift-tube (DT) IM-MS calibration rather than traveling-wave TWIM-MS,
  since the peptide power law parameters and arrival-time-based calibration are specific
  to Waters SYNAPT XS TWIM data.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Calculates calibrated TWCCSN2 values from class-specific power law parameters,
    assigns unknown features to peptide/small-molecule/lipid classes by smallest ΔCCS
    to trendline, and computes assignment scores.
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival time distributions from raw calibration .mzML files and fits
    Gaussian distributions whose apexes become calibrant arrival times feeding MOCCal's
    power law regressions.
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:96036c0560d32ee802a573eee071385990f2fb2eeecfc86f048fae2c8cf51e06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# peptide-feature-class-assignment

## Summary

Assigns unknown TWIM-MS features to the peptide biomolecular class and applies the peptide power law calibration to yield calibrated TWCCSN2 values without prior feature identification.

## When to use

Use when processing traveling-wave IM-MS feature tables (m/z, charge, retention time, arrival time) from multiomic experiments where peptide features such as ADH or enolase digests and polyalanine peptides must receive class-matched CCS calibration without prior identification.

## When NOT to use

- When the feature is a small molecule below m/z 550 — the small-molecule calibration, not the peptide one, should be assigned per the article's m/z constraints.
- When data are negative ionization mode: MOCCal is not yet extended to negative mode data sets.
- When the goal is drift-tube (DT) IM-MS calibration rather than traveling-wave TWIM-MS, since the peptide power law parameters and arrival-time-based calibration are specific to Waters SYNAPT XS TWIM data.

## Inputs

- Experimental feature spreadsheet: feature identifiers, m/z, charge state, retention time, arrival time values
- Class-specific power law calibration parameters (small molecule, lipid, peptide z=1/z=2 fits) from calibrant arrival times and reference DTCCSN2 values
- Peptide DTCCSN2-vs-m/z regression lines from the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)

## Outputs

- Per-feature assigned biomolecular class (peptide) and calibrated TWCCSN2 value
- Assignment score (1–4, or default 0 for multiply charged assignments) reflecting the ΔCCS separation between classes
- Percent difference of calibrated TWCCSN2 against reference drift tube CCS values (e.g. ADH/enolase digest peptides within 2.6%; Poly-Ala6 within 0.6%, Poly-Ala7 within 1.2%)

## How to apply

Import experimental data as a spreadsheet containing feature identifiers, m/z, charge state, retention time, and arrival time values, then use MOCCal to calculate up to three calibrated TWCCSN2 values per feature using small molecule, lipid, and peptide class-specific power law parameters. For each candidate value, compute the ΔCCS against the peptide DTCCSN2-vs-m/z regression line derived from the Unified CCS Compendium; respect the m/z constraint that peptides (like lipids) apply above m/z 300. Select the calibration with the smallest ΔCCS to its trendline; if the assigned class falls outside m/z constraints, the next closest ΔCCS match is chosen. Multiply charged features (e.g. z=2, z=3 polyalanine peptides) receive a default assignment score of 0 and are validated against the peptide trendline rather than small-molecule fits, which is why doubly/triply charged polyalanine calibrated values fall within 0.62% of drift tube reference values.

## Related tools

- **MOCCal** (Calculates calibrated TWCCSN2 values from class-specific power law parameters, assigns unknown features to peptide/small-molecule/lipid classes by smallest ΔCCS to trendline, and computes assignment scores.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival time distributions from raw calibration .mzML files and fits Gaussian distributions whose apexes become calibrant arrival times feeding MOCCal's power law regressions.) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py --calibration_rawdata calibrant.mzML --experimental_features experimental_features.csv
```

## Evaluation signals

- Calibrated TWCCSN2 for doubly/triply charged polyalanine peptides must fall within 0.62% of drift tube values.
- Poly-Ala6 and Poly-Ala7 must reproduce reference CCS within 0.6% and 1.2%, respectively.
- ADH and enolase digest peptides must yield calibrated TWCCSN2 within 2.6% of reference values.
- Every output feature must have exactly one assigned class whose calibrated value minimizes ΔCCS to its m/z-constrained trendline; multiply charged assignments carry a default score of 0.
- Class-assignment accuracy should reach ≥70% for peptides at ±0.5% calibration error, per the article's validation.

## Limitations

- MOCCal is built around small molecule, lipid, and peptide calibrations only — no other endogenous or xenobiotic chemical classes are supported.
- Negative ionization mode data sets are not yet supported; user-driven expansion is welcomed.
- Single power law calibrations introduce calibration biases that may confound feature identification if the peptide class assignment is not applied.

## Evidence

- [results] MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [results] doubly and triply charged polyalanine peptides' calibrated TWCCSN2 values were within 0.62% of drift tube values: [evidence span withheld — anchor status: not_found]
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides"
- [results] Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [discussion] MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
