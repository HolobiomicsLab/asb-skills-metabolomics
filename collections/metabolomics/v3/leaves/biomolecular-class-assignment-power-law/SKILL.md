---
name: biomolecular-class-assignment-power-law
description: Use when processing traveling-wave ion mobility (TWIM-MS) feature tables
  where each feature carries m/z, charge state, retention time, and arrival time but
  no prior identification, and you need an accurate class-specific CCS value (e.g.,
  NIST SRM 1950 extracts spiked with protein digests).
when_to_use_negative:
- Data are already feature-identified (e.g., annotated lipids or peptides with known
  CCS references) — MOCCal is designed for assignment of unknown features without
  prior identification.
- Data are in negative ionization mode, or features belong to endogenous/xenobiotic
  chemical classes outside the small molecule / lipid / peptide scope.
- The acquisition platform is not traveling-wave IM-MS (e.g., drift-tube IM-MS or
  TIMS), since the power-law calibrations and arrival-time-to-TWCCSN2 relationships
  are TWIM-specific.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Python application that executes the class-assignment and power-law CCS calibration
    workflow; available as a pre-built executable (UserDT) or script (RawDT) for unprocessed
    calibration files.
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used in RawDT mode to extract arrival-time distributions from raw calibration
    .mzML files and fit them to Gaussian distributions, recording the apex as each
    calibrant's arrival time prior to power-law optimization.
  repo: http://github.com/pnnl/deimos
- name: Unified CCS Compendium
  role: Source of reference DTCCSN2 values used to optimize class-specific power-law
    parameters and to build the DTCCSN2-vs-m/z trendlines against which ΔCCS is computed.
- name: NIST SRM 1950
  role: Human serum/plasma reference material whose single-phase extract, spiked with
    enolase and alcohol dehydrogenase digests, serves as the multiomic test mixture
    for evaluating class-assignment accuracy.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:ce99fe4f01e5a353968770303a677e0c3aa8231b7a2a11a379ba72f0559e9611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - Agilent ESI-L Tune Mix
  - DEIMoS
  - MOCCal
  - NIST SRM 1950
  - Unified CCS Compendium
  - poly-dl-alanine
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# biomolecular-class-assignment-power-law

## Summary

A method for assigning unannotated TWIM-MS features to small molecule, lipid, or peptide (z=1–3) biomolecular classes by comparing each feature's class-specific power-law-calibrated TWCCSN2 value to m/z-binned class trendlines, then selecting the calibration with the smallest ΔCCS. Developed in MOCCal (Hynds & Hines 2024, Anal. Chem.) for multiomic collision cross section calibration without prior feature identification.

## When to use

Use when processing traveling-wave ion mobility (TWIM-MS) feature tables where each feature carries m/z, charge state, retention time, and arrival time but no prior identification, and you need an accurate class-specific CCS value (e.g., NIST SRM 1950 extracts spiked with protein digests).

## When NOT to use

- Data are already feature-identified (e.g., annotated lipids or peptides with known CCS references) — MOCCal is designed for assignment of unknown features without prior identification.
- Data are in negative ionization mode, or features belong to endogenous/xenobiotic chemical classes outside the small molecule / lipid / peptide scope.
- The acquisition platform is not traveling-wave IM-MS (e.g., drift-tube IM-MS or TIMS), since the power-law calibrations and arrival-time-to-TWCCSN2 relationships are TWIM-specific.

## Inputs

- Experimental feature spreadsheet with columns for feature identifier, m/z, charge state, retention time, and arrival time (drift time).
- Pre-processed calibration data: calibrant arrival times (e.g., poly-dl-alanine, Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix) paired with reference DTCCSN2 values from the Unified CCS Compendium.
- Raw calibration .mzML files when using MOCCal's RawDT mode with DEIMoS to extract arrival-time Gaussian apexes.

## Outputs

- Per-feature assignment to one of the three biomolecular classes (small molecule, lipid, peptide z=1/2/3) with the class-specific calibrated TWCCSN2 value.
- ΔCCS (Å2) values between the calibrated TWCCSN2 and each class trendline, used to identify the smallest-ΔCCS match.
- Per-class calibration error percentages and RMSE values (e.g., 0.02/0.75/0.01/0.06/0.13% errors and 0.28/0.56/0.09/0.09/0.24 RMSE for the NIST SRM 1950 test mixture) reported in the 'Output' folder.

## How to apply

Calibrate every singly charged feature's arrival time with up to three class-specific power-law equations (small molecule, lipid, peptide) whose parameters are optimized from calibrant arrival times and reference DTCCSN2 values from the Unified CCS Compendium. Compute ΔCCS — the difference in Å2 between each calibrated TWCCSN2 value and the corresponding DTCCSN2-vs-m/z trendline — and select the calibrated value with the smallest ΔCCS as the correct class assignment. Enforce m/z constraints during assignment (small molecules < m/z 550, lipids > m/z 300): if a feature falls outside the m/z range for the assigned class, MOCCal falls back to the next closest match by ΔCCS. Charge states matter because multiply charged features (e.g., peptides with z=2 or 3) follow their own class-specific power laws, so a z=1 calibration applied to a z=2 peptide gives substantially larger error; a calibration-effect score (1=<1% to 4=≥6% difference, multiply charged default 0) tracks assignment confidence.

## Related tools

- **MOCCal** (Python application that executes the class-assignment and power-law CCS calibration workflow; available as a pre-built executable (UserDT) or script (RawDT) for unprocessed calibration files.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used in RawDT mode to extract arrival-time distributions from raw calibration .mzML files and fit them to Gaussian distributions, recording the apex as each calibrant's arrival time prior to power-law optimization.) — http://github.com/pnnl/deimos
- **Unified CCS Compendium** (Source of reference DTCCSN2 values used to optimize class-specific power-law parameters and to build the DTCCSN2-vs-m/z trendlines against which ΔCCS is computed.)
- **NIST SRM 1950** (Human serum/plasma reference material whose single-phase extract, spiked with enolase and alcohol dehydrogenase digests, serves as the multiomic test mixture for evaluating class-assignment accuracy.)

## Examples

```
MOCCal.exe --input feature_table.xlsx --calibration polyDL-Ala_WatersMajorMix_AgilentTuneMix.xlsx --output Output/
```

## Evaluation signals

- Per-class calibration errors should reproduce the reported values: 0.02% (lipid), 0.75% (small molecule), and 0.01/0.06/0.13% for peptides at z=1/2/3 on the NIST SRM 1950 mixture.
- RMSE values should match the published table: 0.28, 0.56, 0.09, 0.09, 0.24 for the same classes/charge states.
- Correct assignment rate for lipids and singly charged peptides should be ≥70% when calibration errors are ±0.5% (per the article's assignment-accuracy analysis).
- A feature's assigned class must respect m/z constraints (<550 for small molecules, >300 for lipids); violations should trigger fallback to the next-closest ΔCCS match, which can be inspected in the output.
- Multiply charged features (z>1) should receive calibration-effect score 0 by default; singly charged features should get scores 1–4 corresponding to 1–3%, 3–6%, and ≥6% differences between the assigned and next-closest calibration.

## Limitations

- MOCCal is limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints; other biomolecular classes are not supported out of the box.
- Negative ionization mode data sets and other endogenous or xenobiotic chemical classes are not yet supported; user-driven expansion is welcomed.
- Single power-law calibration (class-agnostic) introduces calibration biases that can confound feature identification — the class-matched assignment is the core value of this method, not a generic calibration.
- A calibration-effect score of 4 (≥6% difference between assigned and next-closest calibration) signals low confidence and may indicate ambiguous class assignment.

## Evidence

- [abstract] Calibrate every singly charged feature's arrival time with up to three class-specific power-law equations: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] Select calibrated TWCCSN2 value with smallest ΔCCS to corresponding trendline: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] m/z constraints enforce class assignment fallback: [evidence span withheld — anchor status: not_found]
- [results] Peptide z=2,3 calibrations differ from z=1: "each biomolecular class has distinct fit parameters for the power law calibration curve"
- [results] ≥70% correct assignment for lipids and singly charged peptides with ±0.5% calibration errors: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [results] Calibration effect score thresholds (1–4) and default score 0 for multiply charged: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match"
- [results] Import experimental data as spreadsheet with feature identifier, m/z, charge state, RT, arrival time: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [results] Per-class calibration errors and RMSE values (0.02/0.75/0.01/0.06/0.13% and 0.28/0.56/0.09/0.09/0.24): [evidence span withheld — anchor status: not_found]
- [discussion] MOCCal limited to small molecule, lipid, and peptide calibrations: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
- [readme] README: MOCCal is a Python application for TWIM-MS multiomic CCS calibration and class assignment without need for identifying features first: "MOCCal, or Multi-Omic CCS Calibrator, is a Python application for high-dimensional, multi-omic traveling-wave ion mobility mass spectrometry (TWIM-MS) data anlaysis. Functionality includes collision"
- [results] DEIMoS extracts arrival times from raw calibration .mzML files and fits Gaussian distributions to record apex: [evidence span withheld — anchor status: not_found]
