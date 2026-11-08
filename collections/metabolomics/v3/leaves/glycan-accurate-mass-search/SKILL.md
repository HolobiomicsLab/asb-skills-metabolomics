---
name: glycan-accurate-mass-search
description: Use when you need to identify N-linked glycans from Orbitrap LC-MS/MS
  (mzXML) or permethylated MALDI-TOF peak lists by matching observed masses to a theoretical
  glycan mass database at a defined ppm tolerance.
when_to_use_negative:
- The glycan of interest is not in the database — the app has no de novo glycan identification
  capability and is limited to glycans already located in the database
- The analytes are O-linked glycans — the app's scope is limited to N-linked glycans
- You need to process multiple mzXML files simultaneously — this is a planned future
  improvement, not implemented
tools:
- name: GlyKAn AZ app
  role: MATLAB app whose MS1/MS2 mass search algorithms and MALDI Peak ID tab perform
    the database mass search and glycan identification
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime platform on which the GlyKAn AZ app (MS1 Peak ID, MS2 Peak Confirmation,
    MALDI Peak ID tabs) is launched
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: ProteoWizard
  role: Converts raw Orbitrap LC-MS/MS files to mzXML format required as input by
    the app's mass search
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:16c0560f8566e9d135d9178d554814ef8f3e672e940ffc2a28dcc4c513f0eea3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MATLAB
  - OrbiTrap
  - ProteoWizard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# glycan-accurate-mass-search

## Summary

Accurate-mass search that matches observed MS1/MS2 peak masses against a glycan theoretical-mass database to identify fluorescently labeled N-linked glycans in Orbitrap LC-MS/MS and permethylated MALDI-TOF data, as implemented in the GlyKAn AZ MATLAB app.

## When to use

Use when you need to identify N-linked glycans from Orbitrap LC-MS/MS (mzXML) or permethylated MALDI-TOF peak lists by matching observed masses to a theoretical glycan mass database at a defined ppm tolerance.

## When NOT to use

- The glycan of interest is not in the database — the app has no de novo glycan identification capability and is limited to glycans already located in the database
- The analytes are O-linked glycans — the app's scope is limited to N-linked glycans
- You need to process multiple mzXML files simultaneously — this is a planned future improvement, not implemented

## Inputs

- mzXML file (raw Orbitrap LC-MS/MS data converted with ProteoWizard)
- MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes)
- MALDI-TOF spectrum file (e.g., permethylated N-glycan run mixed with Super-DHB matrix)
- Glycan mass database (glycan names and theoretical masses in Da) shipped with the app
- User-set thresholds: relative intensity threshold (%) and mass accuracy threshold (ppm)

## Outputs

- MS1 output Excel file listing likely glycans with matched masses and retention times
- MALDI peak-identification output table mapping observed peaks to glycans
- Annotated and labeled MS2 fragmentation spectra for selected glycan-RT pairs (MS2 Peak Confirmation tab)

## How to apply

Convert raw Orbitrap LC-MS/MS files to mzXML with ProteoWizard, then load the mzXML file and the MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes) into the MS1 Peak ID tab of the GlyKAn AZ app and run the MS1 mass search. Use the dataset-optimized thresholds reported in the paper — a 25% relative intensity threshold with a 10 ppm mass accuracy threshold — which yielded 20 true positives and 0 false positives on the example monoclonal antibody dataset while still identifying all 20 manually confirmed glycans. Review the returned MS1 output Excel file to determine which glycans are present, then optionally carry glycan-RT pairs into the MS2 Peak Confirmation tab with the MS2 Fragment Database to obtain annotated and labeled fragmentation spectra. For MALDI-TOF data, load the example MALDI spectrum and glycan mass database into the MALDI Peak ID tab and run the built-in peak-detection and mass-search to match observed peak masses against theoretical masses; false positives can arise from doubly assigned peaks, so verify assignments manually.

## Related tools

- **GlyKAn AZ app** (MATLAB app whose MS1/MS2 mass search algorithms and MALDI Peak ID tab perform the database mass search and glycan identification) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime platform on which the GlyKAn AZ app (MS1 Peak ID, MS2 Peak Confirmation, MALDI Peak ID tabs) is launched) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **ProteoWizard** (Converts raw Orbitrap LC-MS/MS files to mzXML format required as input by the app's mass search)

## Evaluation signals

- On the example mAb LC-MS/MS dataset at 25% relative intensity and 10 ppm mass accuracy, the search returns 20 true positives and 0 false positives
- All true positives are identified at the same retention times as manual analysis
- On the permethylated MALDI-TOF example, the output contains all 49 manually confirmed glycans plus exactly one additional peak that was unassigned during manual analysis
- For MALDI data, the same unidentified peak is labeled with the same glycan at every mass accuracy threshold tested (robustness check)
- The three isomer peaks of equal mass (A2G0M4F / A1G1F(α1,6) / A1G1F(α1,3)) are reported at RTs 13.39, 13.77, and 14.07 min

## Limitations

- No de novo glycan identification: only glycans present in the supplied database can be found
- O-linked glycans are out of scope; the app covers N-linked glycans only
- Cannot process multiple mzXML files simultaneously (future improvement)
- The isomer peak retention time range is a fixed value, not customizable
- MALDI-mode false positives can arise from doubly assigned peaks

## Evidence

- [abstract] MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass: "MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass"
- [intro] The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
- [results] A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [results] All 20 true positives were identified by the app at the same RTs as with manual analysis.: "All 20 true positives were identified by the app at the same RTs as with manual analysis."
- [discussion] The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database.: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
