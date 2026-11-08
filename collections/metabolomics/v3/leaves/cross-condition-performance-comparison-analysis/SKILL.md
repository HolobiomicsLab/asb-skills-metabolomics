---
name: cross-condition-performance-comparison-analysis
description: Use when MOCCal is applied to datasets acquired under altered instrument
  settings (e.g. ramped traveling wave velocity instead of static) and you must quantify
  whether class-assignment rates and calibration errors remain comparable before trusting
  downstream CCS values.
when_to_use_negative:
- Input is a single-condition dataset with no reference condition processed under
  the same MOCCal pipeline — there is nothing to compare
- The goal is feature identification or CCS calibration itself rather than assessing
  robustness across acquisition settings (use the MOCCal calibration skill instead)
- Data are not traveling-wave IM-MS acquisitions (MOCCal is built around TWIM power-law
  calibrations and arrival-time extraction)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MOCCal
  role: Executed identically on both wave-velocity conditions to produce class assignments
    and calibrated TWCCSN2 values for comparison
  repo: https://github.com/HinesLab/MOCCal
- name: Waters SYNAPT XS
  role: TWIM-MS instrument on which the compared datasets (static vs ramped wave velocity)
    were acquired
- name: DEIMoS
  role: Extracts arrival-time data from raw calibrant .mzML files (Gaussian apex)
    so both conditions share identical calibration-curve inputs
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:91d67c72b8d234101272412c781e13365934714cddc53e75ccb263f23c4c0341
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-condition-performance-comparison-analysis

## Summary

A structured side-by-side benchmarking procedure that evaluates whether MOCCal's biomolecular class assignment and TWCCSN2 calibration accuracy are stable across instrument acquisition conditions, e.g. static 600 m/s traveling wave versus ramped wave velocity, using the identical calibration pipeline on each dataset.

## When to use

Use when MOCCal is applied to datasets acquired under altered instrument settings (e.g. ramped traveling wave velocity instead of static) and you must quantify whether class-assignment rates and calibration errors remain comparable before trusting downstream CCS values.

## When NOT to use

- Input is a single-condition dataset with no reference condition processed under the same MOCCal pipeline — there is nothing to compare
- The goal is feature identification or CCS calibration itself rather than assessing robustness across acquisition settings (use the MOCCal calibration skill instead)
- Data are not traveling-wave IM-MS acquisitions (MOCCal is built around TWIM power-law calibrations and arrival-time extraction)

## Inputs

- Experimental feature table for the novel acquisition condition (spreadsheet: feature identifier, m/z, charge state, retention time, arrival time)
- Raw calibrant .mzML files or a processed calibration arrival-time spreadsheet (UserDT mode)
- Reference-condition feature table processed identically (e.g. static 600 m/s dataset)
- Drift-tube CCS reference values from the Unified CCS Compendium / calibration standards

## Outputs

- Side-by-side table of class-assignment rates (%) for small molecules, lipids, and peptides under each wave-velocity condition
- Side-by-side table of calibration errors (% deviation from DTCCSN2) per biomolecular class
- Comparability verdict on whether MOCCal performance is maintained across conditions

## How to apply

Run the identical MOCCal procedure — calibration-curve construction from .mzML calibrant files via DEIMoS Gaussian apex extraction, then class assignment of each singly charged feature under the m/z constraints (<550 for small molecules, >300 for lipids) with the ΔCCS criterion — separately on the novel-condition dataset and on the reference dataset (e.g. static 600 m/s). Tabulate class-assignment percentages (small molecule, lipid, peptide) and per-feature calibration error (percent deviation of calibrated TWCCSN2 from DTCCSN2) for both conditions side by side. Compare errors and assignment rates across conditions, using the static-condition reference benchmarks of ≥70% correct assignment for lipids/peptides and ±0.5–1% typical calibration errors as comparability anchors. Judge comparability, not absolute performance: the skill's core rationale is that acquisition-parameter changes (wave velocity) should not systematically shift the power-law fit parameters or the ΔCCS ranking of the three class calibrations.

## Related tools

- **MOCCal** (Executed identically on both wave-velocity conditions to produce class assignments and calibrated TWCCSN2 values for comparison) — https://github.com/HinesLab/MOCCal
- **Waters SYNAPT XS** (TWIM-MS instrument on which the compared datasets (static vs ramped wave velocity) were acquired)
- **DEIMoS** (Extracts arrival-time data from raw calibrant .mzML files (Gaussian apex) so both conditions share identical calibration-curve inputs) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run once per condition (ramped vs static 600 m/s); install DEIMoS v1.3.2 first (http://github.com/pnnl/deimos)
```

## Evaluation signals

- Ramped-condition class-assignment rates for lipids and peptides should meet the static-condition benchmark of ≥70% correct assignment
- Calibration errors under the ramped condition should remain within ±1% of DTCCSN2 values, matching the static 600 m/s condition
- Both conditions must have used the same three class-specific power-law regressions and the same ΔCCS (Å²) best-curve selection criterion, so differences are attributable to acquisition, not pipeline drift
- Per-class assignment percentages (e.g. 81–83% for CCS Compendium small molecules) should be tabulated side by side rather than pooled across conditions
- Any observed error shift should be checked against the calibration effect-score scale (1 = <1%, 4 = ≥6%) to verify it stays at the low-score end

## Limitations

- MOCCal's calibrations are limited to small molecule, lipid, and peptide classes; other endogenous or xenobiotic chemical classes are out of scope
- Not yet extended to negative ionization mode datasets
- Comparison anchors are defined by the paper's static 600 m/s condition; other reference conditions would require re-benchmarking before comparability claims

## Evidence

- [readme] Ramped-condition class-assignment rates for lipids and peptides should meet the static-condition benchmark of ≥70% correct assignment: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] Calibration errors under the ramped condition should remain within ±1% of DTCCSN2 values, matching the static 600 m/s condition: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
