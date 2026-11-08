---
name: lipid-feature-annotation-against-lipidpioneer
description: Use when lipid features from traveling-wave IM-MS (m/z, charge, retention
  time, arrival time) have been class-assigned and CCS-calibrated and you need database-backed
  validation of exact mass and CCS against curated lipid reference values.
when_to_use_negative:
- Features have not yet been CCS-calibrated or class-assigned by MOCCal (the skill
  assumes a MOCCal output with class, TWCCSN2, and ΔCCS columns)
- Data are in negative ionization mode or from non-TWIM platforms — MOCCal and its
  calibrations are currently limited to small molecule, lipid, and peptide classes
  on TWIM-MS instruments like the Waters SYNAPT XS
tools:
- name: MOCCal
  role: Performs class assignment and class-specific TWCCSN2 calibration whose lipid-assigned
    features are then validated against LipidPioneer
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival-time distributions from raw .mzML calibration files and fits
    Gaussians to obtain apex arrival times used in the calibration and subsequent
    annotation validation
  repo: http://github.com/pnnl/deimos
- name: LipidPioneer
  role: Reference database of lipid exact masses and CCS values used to evaluate feature
    annotations
- name: CCS Compendium
  role: Reference database of DTCCSN2 values against which calibrated TWCCSN2 errors
    are computed
- name: CCSBase
  role: CCS resource context for CCS calibration and prediction tools discussed alongside
    MOCCal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:20d1142475f07cf0109ff4026dfc1a5cd607c44fd4452d5952d6d7d31180c89f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCS Compendium
  - CCSBase
  - DEIMoS
  - LipidPioneer
  - MOCCal
  - Waters AQUITY UPLC BEH amide
  - Waters SYNAPT XS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lipid-feature-annotation-against-lipidpioneer

## Summary

Validates MOCCal-classified lipid features from TWIM-MS (e.g., HILIC-IM-MS of NIST SRM 1950 plasma extract) by comparing exact masses and calibrated TWCCSN2 values against the LipidPioneer and Unified CCS Compendium databases to quantify per-feature assignment error and cross-class misassignment.

## When to use

Use when lipid features from traveling-wave IM-MS (m/z, charge, retention time, arrival time) have been class-assigned and CCS-calibrated and you need database-backed validation of exact mass and CCS against curated lipid reference values.

## When NOT to use

- Features have not yet been CCS-calibrated or class-assigned by MOCCal (the skill assumes a MOCCal output with class, TWCCSN2, and ΔCCS columns)
- Data are in negative ionization mode or from non-TWIM platforms — MOCCal and its calibrations are currently limited to small molecule, lipid, and peptide classes on TWIM-MS instruments like the Waters SYNAPT XS

## Inputs

- MOCCal output table of assigned class, calibrated TWCCSN2, ΔCCS, and calibration-effect score for each feature
- Feature spreadsheet with feature identifier, m/z, charge state, retention time, and arrival time (e.g., from HILIC-IM-MS analysis of NIST SRM 1950 single-phase extract)
- LipidPioneer database of lipid exact masses and CCS values
- Unified CCS Compendium DTCCSN2 reference values

## Outputs

- Per-feature annotation table with lipid identity candidate, exact mass match, reference CCS, and percent error vs calibrated TWCCSN2
- Per-class assignment rate (e.g., 88.6% of 44 lipid features correctly assigned) and calibration-effect scores (1–4)
- List of cross-class misassignments (e.g., trazodone via lipid calibration; lysophospholipids via peptide calibration)

## How to apply

After MOCCal calibrates each singly charged feature with up to three class-specific power law equations (small molecule, lipid, peptide; z = 1) and selects the calibration with the smallest ΔCCS to the DTCCSN2-vs-m/z regression trendlines, take the lipid-assigned features (m/z > 300) and evaluate their exact masses and calibrated TWCCSN2 values against LipidPioneer and CCS Compendium database entries. Compute the percent error of the calibrated TWCCSN2 relative to the reference DTCCSN2 (e.g., 0.18% for a lipid matched by lipid-fit power law vs 1.8–2.6% under small-molecule or peptide calibrations), and record the calibration-effect score (1–4, where score 1 = <1% and score 4 = ≥6% difference to the next closest match). Flag cross-class misassignments, such as trazodone (m/z 372.159) assigned via the lipid calibration with ~3.2% error or lysophospholipids calibrated by the peptide equation (0.5–1.8% error). Aggregate per-class assignment rates and RMSE values (lipid RMSE ~0.28 Å2) as the final performance metrics.

## Related tools

- **MOCCal** (Performs class assignment and class-specific TWCCSN2 calibration whose lipid-assigned features are then validated against LipidPioneer) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival-time distributions from raw .mzML calibration files and fits Gaussians to obtain apex arrival times used in the calibration and subsequent annotation validation) — http://github.com/pnnl/deimos
- **LipidPioneer** (Reference database of lipid exact masses and CCS values used to evaluate feature annotations)
- **CCS Compendium** (Reference database of DTCCSN2 values against which calibrated TWCCSN2 errors are computed)
- **CCSBase** (CCS resource context for CCS calibration and prediction tools discussed alongside MOCCal)

## Evaluation signals

- ≥88.6% of the 44 lipid features in the NIST SRM 1950 extract are assigned to the lipid calibration (≥93.3% of 15 small-molecule features to the small-molecule calibration)
- Per-class RMSE of calibrated TWCCSN2 vs reference values matches ~0.28 Å2 (lipid), ~0.58 Å2 (small molecule), ~0.12 Å2 (peptide)
- The lipid feature at m/z 692.517 calibrated with the lipid power law yields ~0.18% error vs DTCCSN2, while the same feature under small-molecule/peptide calibrations gives 1.8%/2.6% — confirming smallest-ΔCCS selection
- Known cross-class cases reproduce: trazodone (m/z 372.159) via lipid calibration shows ~3.2% error; lysophospholipids via the peptide calibration show 0.5–1.8% error
- Calibration-effect scores fall in the defined 1–4 range (1 = <1%, 2 = 1–3%, 3 = 3–6%, 4 = ≥6% difference to next closest match), with multiply charged assignments defaulting to score 0

## Limitations

- MOCCal currently supports only small molecule, lipid, and peptide calibrations within TWIM-MS constraints
- MOCCal is not yet extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Single power law calibrations can introduce calibration biases that confound or prohibit feature identification, which is why class-specific calibration and per-feature database validation are needed

## Evidence

- [results] annotation validation against databases: "evaluated the exact masses and CCS values of the features against the LipidPioneer and CCS Compendium databases"
- [results] regression reference source: "regression lines developed from DTCCSN2 values in the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)"
- [results] selection rule: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] score thresholds: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match"
- [readme] software scope and usage: "Functionality includes collision cross section (CCS) calibration, experimental data biomolecular class assignment, and experimental class-specific CCS calculations."
