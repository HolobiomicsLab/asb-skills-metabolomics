---
name: ms1-accurate-mass-glycan-searching
description: Use when you have LC-MS/MS data of fluorescent-labeled N-linked glycans
  (OrbiTrap mzXML) plus a list of candidate glycan names and theoretical masses, and
  need to find which glycans are present at which retention times.
when_to_use_negative:
- The sample contains O-linked glycans or de novo unknown glycans not already in the
  database — the app's scope is limited to N-linked glycans present in the MS1 Database.
- You need to process multiple mzXML files simultaneously — this is a planned future
  improvement and is not implemented.
- You already have MS2 fragmentation data and need spectral annotation rather than
  MS1 peak identification — use MS2 Peak Confirmation instead.
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ
  role: Hosts the MS1 search algorithm that matches glycan theoretical masses to peaks
    in the mzXML and returns the MS1 output Excel file.
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Underlying platform on which GlyKAn AZ's MS1 search is implemented.
- name: ProteoWizard
  role: Converts raw MS files to mzXML format, which is the required input for the
    MS1 search.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:dc2a6ba8a9a9f423c600770d7bdd381dc438541d09c41d9db3e7dfdcb909d176
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
  tools:
  - GlyKAn AZ
  - MATLAB
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# MS1 accurate-mass glycan searching

## Summary

An MS1-level search that matches theoretical glycan masses in an MS1 Database against peaks in an mzXML file within a user-specified mass-accuracy tolerance (e.g. 10 ppm), to identify likely fluorescent-labeled N-linked glycans and their isomers prior to MS2 confirmation.

## When to use

Use when you have LC-MS/MS data of fluorescent-labeled N-linked glycans (OrbiTrap mzXML) plus a list of candidate glycan names and theoretical masses, and need to find which glycans are present at which retention times.

## When NOT to use

- The sample contains O-linked glycans or de novo unknown glycans not already in the database — the app's scope is limited to N-linked glycans present in the MS1 Database.
- You need to process multiple mzXML files simultaneously — this is a planned future improvement and is not implemented.
- You already have MS2 fragmentation data and need spectral annotation rather than MS1 peak identification — use MS2 Peak Confirmation instead.

## Inputs

- mzXML file (converted from raw OrbiTrap MS data via ProteoWizard)
- MS1 Database containing glycan names, theoretical masses in daltons, and optionally expected RTs in minutes
- User-specified mass-accuracy tolerance (ppm) and relative intensity threshold

## Outputs

- MS1 output Excel file listing glycan name, charge state, theoretical mass, observed m/z, mass error, retention time, and intensity
- List of confirmed glycan–RT pairs, including isomer peaks, for downstream MS2 Peak Confirmation

## How to apply

Load the mzXML file and an MS1 Database listing glycan names, theoretical masses in daltons, and optionally expected RTs in minutes into the MS1 tab of GlyKAn AZ. The algorithm loops over each glycan and each allowed charge state, scanning the MS1 spectra for the highest-intensity peak matching the theoretical mass within the user-specified mass-accuracy tolerance — 10 ppm gave the optimal tradeoff in the paper's evaluation. Isomers at the same m/z within a fixed ±1.75 min retention-time window are called by walking the extracted ion chromatogram and flagging points where 3-of-4 consecutive intensity slopes are positive followed by 3-of-4 consecutive slopes negative, then filtering with a 25% relative-intensity threshold. Output is an MS1 Excel file listing glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity, which is reviewed to decide which glycan–RT pairs to carry into MS2 Peak Confirmation.

## Related tools

- **GlyKAn AZ** (Hosts the MS1 search algorithm that matches glycan theoretical masses to peaks in the mzXML and returns the MS1 output Excel file.) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Underlying platform on which GlyKAn AZ's MS1 search is implemented.)
- **ProteoWizard** (Converts raw MS files to mzXML format, which is the required input for the MS1 search.)

## Evaluation signals

- The output Excel file should contain the expected glycan names, charge states, observed m/z values, mass errors within the specified tolerance (e.g. ≤10 ppm), retention times, and intensities.
- At the paper's optimum (25% relative intensity threshold, 10 ppm mass accuracy), the search should recover all manually confirmed glycans (20 true positives) with minimal false positives (0 in the paper's dataset).
- Every reported glycan peak must have a mass error within the user-specified ppm tolerance.
- Isomer peaks must appear at the same m/z within ±1.75 min of the representative peak's RT and pass the 25% relative-intensity filter.
- Identified RTs should match those from manual analyst review when run on a dataset with manually assigned glycans.

## Limitations

- The isomer peak retention-time range is a fixed value (±1.75 min) and is not customizable by the user.
- The app cannot process multiple mzXML files simultaneously; batch processing is a planned future improvement.
- No de novo glycan identification is available — only glycans already present in the MS1 Database can be found.
- The scope is limited to N-linked glycans; O-linked glycan analysis is not supported.
- Requires a user-supplied mass-accuracy tolerance; an overly loose tolerance (e.g. 50 ppm) introduces false positives while an overly strict one (e.g. 100% relative threshold) misses true positives.

## Evidence

- [intro] The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app, and MS1: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app, and MS1"
- [results] A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [discussion] including the isomer peak retention time range, which is currently a fixed value that meets most use cases.: "including the isomer peak retention time range, which is currently a fixed value that meets most use cases."
- [abstract] MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass: "MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass"
- [intro] The MS1 output Excel file that the app returns can then be reviewed to determine which glycans are likely to be present in the sample.: "The MS1 output Excel file that the app returns can then be reviewed to determine which glycans are likely to be present in the sample."
