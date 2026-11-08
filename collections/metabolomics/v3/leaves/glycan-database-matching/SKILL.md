---
name: glycan-database-matching
description: Use when confirming fluorescent-labeled or permethylated N-linked glycan
  species from LC–MS/MS or MALDI–TOF peak lists by accurate-mass search against a
  curated glycan mass database with defined ppm and relative-intensity thresholds.
when_to_use_negative:
- The glycan of interest is not already in the database — the app has no de novo glycan
  identification capability and is limited to database entries
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans
- Multiple mzXML files need batch matching in one run — the app cannot process multiple
  mzXML files simultaneously
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ app
  role: MATLAB app whose MS1 and MS2 tabs run the glycan database mass search, threshold
    filtering, and fragment annotation
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: runtime environment in which the GlyKAn AZ matching and annotation algorithms
    execute
- name: ProteoWizard
  role: converts raw MS files to mzXML format prior to loading into the MS1 tab for
    mass search
- name: MALDI-TOF
  role: instrument producing permethylated N-glycan spectra that are mass-searched
    against the glycan database in the MALDI Peak ID tab
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:39588d359a9511589751fdc092574a52abcf3593e2b01a800cbb0b40d39d9fc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MALDI-TOF
  - MATLAB
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# glycan-database-matching

## Summary

Mass-search based glycan identification in which observed MS1/MS2 peak masses (mzXML from Orbitrap LC–MS/MS or permethylated MALDI–TOF spectra) are matched against glycan databases of theoretical masses (Da) to confirm N-linked glycan species.

## When to use

Use when confirming fluorescent-labeled or permethylated N-linked glycan species from LC–MS/MS or MALDI–TOF peak lists by accurate-mass search against a curated glycan mass database with defined ppm and relative-intensity thresholds.

## When NOT to use

- The glycan of interest is not already in the database — the app has no de novo glycan identification capability and is limited to database entries
- The sample contains O-linked glycans — the app's scope is limited to N-linked glycans
- Multiple mzXML files need batch matching in one run — the app cannot process multiple mzXML files simultaneously

## Inputs

- mzXML file converted from raw Orbitrap MS data using ProteoWizard
- MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes)
- MS2 Fragment Database and MS2 fragment image repository
- permethylated N-glycan MALDI-TOF spectrum file and accompanying glycan mass database
- MS1 output Excel file listing candidate glycan-RT pairs
- user-defined mass-accuracy tolerance (ppm) and relative-intensity threshold (%)

## Outputs

- MS1 output Excel file of matched glycan peaks with RTs and true/false-positive classifications
- annotated and labeled MS2 fragmentation spectra for selected glycan-RT pairs
- ms2_annotated_spectra.xlsx with m/z-to-fragment annotations (up to two candidate fragments per m/z, including +Na, +K, -OH adduct forms)

## How to apply

Load the MS data (mzXML converted from raw files via ProteoWizard, or an exported MALDI-TOF spectrum) together with the glycan mass database containing glycan names and theoretical masses in Da into the matching tab of the GlyKAn AZ MATLAB app. Run the MS1 mass search, sweeping relative-intensity and mass-accuracy thresholds — e.g., for the evaluated monoclonal antibody data set a 25% relative-intensity threshold with 10 ppm mass accuracy gave the optimal result (20 true positives, 0 false positives), while 50 ppm at the same intensity threshold inflated false positives to 8. Filter MS2 fragment matches by requiring the relative intensity to exceed the user-set threshold (3% in the reported MS2 annotation of A1G1F(α1,3) at RT 13.77 min) and the fragment mass accuracy to fall within the ppm tolerance, annotating each m/z with up to two candidate fragments including +Na, +K and −OH adduct forms. Export the peak-identification output table (MS1 output Excel file) and compare against manual reference assignments to quantify true and false positives at each threshold setting. For MALDI-TOF data, the validation criterion is that the app recovers all manually confirmed glycans (49 in the permethylated N-glycan example) plus any peaks unassigned manually, with false positives arising only from doubly assigned peaks.

## Related tools

- **GlyKAn AZ app** (MATLAB app whose MS1 and MS2 tabs run the glycan database mass search, threshold filtering, and fragment annotation) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (runtime environment in which the GlyKAn AZ matching and annotation algorithms execute)
- **ProteoWizard** (converts raw MS files to mzXML format prior to loading into the MS1 tab for mass search)
- **MALDI-TOF** (instrument producing permethylated N-glycan spectra that are mass-searched against the glycan database in the MALDI Peak ID tab)

## Examples

```
matlab -batch "apptool('GlyKAn AZ'); % MALDI Peak ID tab: load example MALDI-TOF spectrum + glycan mass database from github.com/ZaylaSchaeffer/GlyKAn-AZ-application and run mass search"
```

## Evaluation signals

- At the chosen threshold combination (e.g., 25% relative intensity, 10 ppm), the output table contains all manually confirmed glycans (20 for the LC–MS/MS mAb data set; 49 for the permethylated MALDI-TOF set) with few or zero false positives
- All true positives are identified at the same retention times as manual analysis (e.g., the three A2G0M4F / A1G1F(α1,6) / A1G1F(α1,3) isomer peaks at RT 13.39, 13.77, 14.07 min)
- Every MS2 m/z annotation carries at most two candidate fragments from the parent glycan, including adduct forms (+Na, +K, -OH), with fragment relative intensity above the 3% threshold and within the ppm tolerance
- MALDI validation: the app labels the same unidentified peak with the same glycan at every mass-accuracy threshold tested, matching the manual reference where the peak was unassigned
- Exported ms2_annotated_spectra.xlsx is populated and renderable via the MS2 fragment image repository

## Limitations

- No de novo glycan identification capability; identification is limited to glycans already located in the database
- O-linked glycans are not supported; scope is restricted to N-linked glycans
- The app cannot process multiple mzXML files simultaneously (planned future improvement)
- False positives can arise from doubly assigned peaks in MALDI data
- Isomer peak retention time range is a fixed value, not customizable

## Evidence

- [abstract] MS1 and MS2 mass search algorithms with glycan databases to confirm fluorescent labeled N-linked glycan species based on accurate mass: "MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass"
- [results] Optimal MS1 thresholds for the mAb data set: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [results] MS2 fragment matching criteria: "of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated"
- [intro] mzXML and MS1 Database loaded into the MS1 tab: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
- [abstract] MALDI-TOF validation of database matching: "matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data and was successfully validated by identifying all glycan species that were previously identified manually"
- [discussion] No de novo identification; database-limited scope: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
- [readme] Repository availability of the app and databases: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan 
identification process from LC-MS/MS data."
