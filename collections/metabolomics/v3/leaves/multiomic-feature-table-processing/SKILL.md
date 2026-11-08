---
name: multiomic-feature-table-processing
description: Use when you have singly charged features from Waters SYNAPT XS traveling-wave
  IM-MS data spanning small molecules, lipids, and peptides, and need class-assigned
  calibrated CCS values without prior identification.
when_to_use_negative:
- Data are from drift-tube IM-MS or non-traveling-wave IM platforms — MOCCal's power
  law calibrations are TWIM-specific.
- Features are multiply charged (charge > 1) — class assignment and ΔCCS-based selection
  operate on singly charged features; multiply charged assignments default to score
  0.
- The analysis targets negative ionization mode or chemical classes beyond small molecules,
  lipids, and peptides — MOCCal has not yet been extended to those.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Executes class-specific power law calibration, biomolecular class assignment,
    and class-specific CCS calculation from the feature table
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival time data from raw .mzML calibration files and fits Gaussian
    distributions to record apex arrival times (RawDT mode); version 1.3.2 required
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Instrument that generates the traveling-wave IM-MS data this skill operates
    on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:d455539d22f9fa5c1b7f2d61cb88f8e9204e45c3c535e387fa998c0b2da0c94d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multiomic-feature-table-processing

## Summary

Prepare a multiomic TWIM-MS feature table (feature IDs, m/z, charge state, retention time, arrival time) and run it through MOCCal's class-specific power law CCS calibrations to assign biomolecular class and calibrated TWCCSN2 values.

## When to use

Use when you have singly charged features from Waters SYNAPT XS traveling-wave IM-MS data spanning small molecules, lipids, and peptides, and need class-assigned calibrated CCS values without prior identification.

## When NOT to use

- Data are from drift-tube IM-MS or non-traveling-wave IM platforms — MOCCal's power law calibrations are TWIM-specific.
- Features are multiply charged (charge > 1) — class assignment and ΔCCS-based selection operate on singly charged features; multiply charged assignments default to score 0.
- The analysis targets negative ionization mode or chemical classes beyond small molecules, lipids, and peptides — MOCCal has not yet been extended to those.

## Inputs

- Experimental feature table as spreadsheet: feature identifier, m/z, charge state, retention time, arrival time
- Raw calibration data as .mzML files (for RawDT mode) or processed calibrant arrival times with reference drift tube CCS values (UserDT mode)

## Outputs

- Class-assigned calibrated TWCCSN2 value per feature
- Biomolecular class assignment (small molecule, lipid, or peptide) per feature
- Calibration effect score (1–4) per assignment

## How to apply

Format experimental data as a spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time; raw calibrant files must be .mzML processed via DEIMoS (v1.3.2) with Gaussian-distributed arrival time apexes. Calibrate every singly charged arrival time with up to three class-specific power law equations (small molecule, lipid, peptide), applying the m/z gates (<550 for small molecules, >300 for lipids). For each candidate calibration compute ΔCCS = calibrated TWCCSN2 minus the DTCCSN2-vs-m/z regression line for that class; select the calibration with the smallest ΔCCS. The rationale is that each biomolecular class has distinct power law fit parameters, and the ΔCCS criterion captures how well each calibration reproduces the class trendline at that feature's m/z.

## Related tools

- **MOCCal** (Executes class-specific power law calibration, biomolecular class assignment, and class-specific CCS calculation from the feature table) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival time data from raw .mzML calibration files and fits Gaussian distributions to record apex arrival times (RawDT mode); version 1.3.2 required) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Instrument that generates the traveling-wave IM-MS data this skill operates on)

## Examples

```
python MOCCal_RawDT.py  # run in DEIMoS v1.3.2 virtualenv on .mzML calibrants + feature spreadsheet (UserDT: MOCCal.exe with Output folder)
```

## Evaluation signals

- For a known lipid (e.g. m/z 692.517), lipid-parametrized calibration should yield TWCCSN2 within ~0.18% of the DTCCSN2 reference, while small molecule/peptide calibrations should show 1.8–2.6% error — confirming class-assignment specificity.
- Class-assignment rates should land at ≥70% correct for lipids and singly charged peptides, with calibration errors within ±0.5% for the best assignments.
- Small molecules from the CCS Compendium should be assigned to small molecule calibrations ~81–83% with errors of ±0.5 to 5.0%.
- Output must include a calibration effect score (1–4) reflecting the difference between the assigned-class calibrated CCS and the next closest match; multiply charged features default to score 0.
- On the NIST SRM 1950 test extract, calibrated features should reproduce reported trends against LipidPioneer and CCS Compendium databases.

## Limitations

- Class assignment assumes the feature's m/z falls within class gates (<550 for small molecules, >300 for lipids); out-of-range features fall back to next-closest ΔCCS match, which can misassign.
- Calibration biases from a single power law equation can confound feature identification if class-specific calibration is not applied.
- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints; negative ionization mode and other endogenous/xenobiotic classes are not yet supported.

## Evidence

- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [abstract] uses the difference (in Å2) between the calibrated TWCCSN2 value and DTCCSN2 vs m/z regression lines to determine the best calibration curve: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [results] raw data for each calibration solution should be loaded as a .mzML file; DEIMoS extracts and Gaussian-fits arrival times: "raw data for each calibration solution should be loaded as a .mzML file"
- [results] Parameters are optimized for class-specific power law regressions using the calibrant's arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [readme] MOCCal https://github.com/HinesLab/MOCCal (accessed MM/YYYY): "MOCCal https://github.com/HinesLab/MOCCal (accessed MM/YYYY)"
