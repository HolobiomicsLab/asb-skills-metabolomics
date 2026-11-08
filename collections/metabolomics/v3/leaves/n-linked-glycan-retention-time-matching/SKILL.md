---
name: n-linked-glycan-retention-time-matching
description: Use when LC-MS/MS data from released, labeled N-glycans (mzXML from Orbitrap
  or MALDI-TOF instruments) must be identified and validated against a reference set
  of manually annotated glycan-RT pairs.
when_to_use_negative:
- The glycans of interest are O-linked — the app's scope is limited to N-linked glycans
- De novo identification of glycans absent from the database is required — the app
  only finds database entries
- Multiple mzXML files must be processed simultaneously — not yet supported
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: GlyKAn AZ
  role: MATLAB app performing MS1 peak identification (mass + RT matching) and MS2
    peak confirmation
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: ProteoWizard
  role: Converts raw Orbitrap MS files to mzXML format required by the app
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ app executes
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:b4414128834eb35cd2e2d561939576172fdf6529c3f4bdbb2ee63fd1518541cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
  tools:
  - GlyKAn AZ
  - MATLAB
  - OrbiTrap
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# N-linked glycan retention-time matching

## Summary

Confirms fluorescent-labeled N-linked glycan identifications by matching LC-MS/MS-detected glycan peaks (mass + retention time) against expected RTs and against manually curated reference identifications, as implemented in the MS1/MS2 tabs of the GlyKAn AZ MATLAB app.

## When to use

Use when LC-MS/MS data from released, labeled N-glycans (mzXML from Orbitrap or MALDI-TOF instruments) must be identified and validated against a reference set of manually annotated glycan-RT pairs.

## When NOT to use

- The glycans of interest are O-linked — the app's scope is limited to N-linked glycans
- De novo identification of glycans absent from the database is required — the app only finds database entries
- Multiple mzXML files must be processed simultaneously — not yet supported

## Inputs

- mzXML file of LC-MS/MS runs (converted from raw Orbitrap data via ProteoWizard)
- MS1 Database: glycan names, theoretical masses (Da), optional expected RTs (minutes)
- MS2 Fragment Database and MS2 fragment image repository
- Manually identified glycan list with retention times (validation reference)

## Outputs

- MS1 output Excel file of candidate glycans with matched masses and RTs
- Annotated and labeled MS2 fragmentation spectra for selected glycan-RT pairs
- True positive / false positive counts against the manual reference (e.g. 20 TP, 0 FP at 25%/10 ppm)

## How to apply

Convert raw Orbitrap LC-MS/MS files to mzXML with ProteoWizard, then load the mzXML file and an MS1 Database (glycan names, theoretical masses in Da, and optionally expected RTs in minutes) into GlyKAn AZ's MS1 tab. Run the MS1 mass search with dataset-tuned thresholds — for the mAb example dataset, a 25% relative intensity threshold and 10 ppm mass accuracy threshold gave 20 true positives and 0 false positives. Review the MS1 output Excel file to decide which glycans are present, then feed it together with the MS2 Fragment Database and fragment image repository into the MS2 Peak Confirmation tab to generate annotated, labeled MS2 spectra for the selected glycan-RT pairs. Validate by checking that app-identified glycans and their retention times coincide with the manual reference: in the published validation all 20 true positives were found at the same RTs as manual analysis.

## Related tools

- **GlyKAn AZ** (MATLAB app performing MS1 peak identification (mass + RT matching) and MS2 peak confirmation) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **ProteoWizard** (Converts raw Orbitrap MS files to mzXML format required by the app)
- **MATLAB** (Runtime environment in which the GlyKAn AZ app executes) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application

## Evaluation signals

- All manually confirmed glycans are recovered as true positives (20/20 in the published mAb validation)
- Zero or minimal false positives at the chosen thresholds (0 FP at 25% intensity / 10 ppm; 3 FP at 50 ppm)
- App-identified retention times match the manual analysis RTs exactly (e.g. A2G0M4F isomers at 13.39, 13.77, 14.07 min)
- MS2 confirmation returns annotated spectra for the selected glycan-RT pairs (e.g. 18 of 24 peaks above 3% relative abundance annotated for A1G1F(α1,3) at RT 13.77 min)

## Limitations

- Threshold choice trades sensitivity for specificity: at 100% relative intensity only 6 TP with 6 FP were found; loosening mass accuracy to 50 ppm adds false positives
- Isomer peak retention time range is a fixed value, not customizable
- No de novo glycan identification; only glycans present in the supplied databases can be found
- Cannot process multiple mzXML files simultaneously (planned future improvement)

## Evidence

- [intro] ms1_optional_rt: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab"
- [results] optimal_thresholds: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [results] rt_match: "All 20 true positives were identified by the app at the same RTs as with manual analysis."
- [results] isomer_rts: "three isomer peaks were identified at RTs of 13.39, 13.77, and 14.07 min"
- [intro] proteowizard: "ProteoWizard is used to convert the raw MS files to mzXML format."
- [readme] readme_scope: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
