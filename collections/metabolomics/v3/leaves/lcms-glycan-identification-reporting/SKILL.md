---
name: lcms-glycan-identification-reporting
description: Use when the goal is to identify fluorescently labeled N-linked glycans
  from LC-MS/MS data acquired on an Orbitrap (or MALDI-TOF) instrument, replacing
  manual analyst peak picking with automated MS1/MS2 searching and Excel reporting.
when_to_use_negative:
- The input data are O-linked glycans or the analysis requires de novo glycan identification
  for structures not in the database — the app is limited to N-linked glycans present
  in its database
- You need to process multiple mzXML files simultaneously — batch processing is a
  planned future improvement, not implemented
- You need a customizable isomer retention-time window — the ±1.75 min isomer RT range
  is fixed and cannot be adjusted
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ
  role: MATLAB-based app that runs the MS1 peak search, isomer detection, MS2 peak
    confirmation, and Excel reporting for N-linked glycan identification
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Programming environment in which the GlyKAn AZ app is implemented and run
- name: ProteoWizard
  role: Converts raw MS files to the mzXML format required by GlyKAn AZ
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:72b989f19443bc8dd19c921ad74b04a4dc98ce65d13480b71a288beb25659446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
  tools:
  - GlyKAn AZ
  - MATLAB
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lcms-glycan-identification-reporting

## Summary

Automate N-linked glycan identification from Orbitrap LC–MS/MS data with GlyKAn AZ: run the MS1 peak search over an mzXML file and MS1 glycan database, detect same-m/z isomer peaks within a ±1.75 min window, confirm candidates by annotated MS2 fragmentation spectra, and report the results in Excel.

## When to use

Use when the goal is to identify fluorescently labeled N-linked glycans from LC-MS/MS data acquired on an Orbitrap (or MALDI-TOF) instrument, replacing manual analyst peak picking with automated MS1/MS2 searching and Excel reporting.

## When NOT to use

- The input data are O-linked glycans or the analysis requires de novo glycan identification for structures not in the database — the app is limited to N-linked glycans present in its database
- You need to process multiple mzXML files simultaneously — batch processing is a planned future improvement, not implemented
- You need a customizable isomer retention-time window — the ±1.75 min isomer RT range is fixed and cannot be adjusted

## Inputs

- Raw Orbitrap LC-MS/MS file converted to mzXML format (via ProteoWizard)
- MS1 Database: glycan names, theoretical masses (Da), optionally expected RTs (minutes)
- MS2 Fragment Database
- MS2 fragment image repository
- User-specified mass accuracy threshold (ppm) and relative intensity threshold (%)

## Outputs

- MS1 output Excel file listing glycan name, charge state, theoretical mass, observed m/z, mass error, retention time, and intensity
- Annotated and labeled MS2 fragmentation spectra for the selected glycan-RT pairs

## How to apply

Convert the raw MS file to mzXML with ProteoWizard, then load the mzXML file and an MS1 Database (glycan names, theoretical masses in Da, optionally expected RTs in minutes) into the GlyKAn AZ MS1 tab. For each glycan and charge state, the app scans MS1 spectra for the highest-intensity peak within the mass-accuracy tolerance — 10 ppm was optimal in the paper — and finds isomers at the same m/z within a fixed ±1.75 min RT window using a slope criterion (3-of-4 consecutive positive intensity points followed by 3-of-4 negative points), filtering isomers with a relative intensity threshold (25% was optimal, yielding 20 true positives / 0 false positives for the evaluated dataset). Review the returned MS1 output Excel file to decide which glycans are likely present, then load that file, the MS2 Fragment Database, and the MS2 fragment image repository into the MS2 Peak Confirmation tab and select glycan-RT pairs for MS2 analysis; the app returns annotated and labeled MS2 fragmentation spectra used for glycan confirmation. New glycans not in the shipped databases can be added via the Fragment Generator, which automatically identifies fragmentation patterns.

## Related tools

- **GlyKAn AZ** (MATLAB-based app that runs the MS1 peak search, isomer detection, MS2 peak confirmation, and Excel reporting for N-linked glycan identification) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Programming environment in which the GlyKAn AZ app is implemented and run)
- **ProteoWizard** (Converts raw MS files to the mzXML format required by GlyKAn AZ)

## Examples

```
matlab -batch "run GKA_app; load mzXML & MS1 Database in the MS1 tab; set 10 ppm & 25% thresholds; run MS1 search; review MS1 output Excel; load into MS2 tab"
```

## Evaluation signals

- All manually confirmed glycans are recovered: the app should identify the same glycan-RT pairs as manual analysis (paper benchmark: all 20 true positives at the same RTs)
- False positives minimized: at 25% relative intensity and 10 ppm mass accuracy the paper reported 20 true positives / 0 false positives; deviations suggest the thresholds need tuning
- MS2 annotation coverage: for a validated glycan such as A1G1F(α1,3) at RT 13.77 min, 18 of 24 distinct fragment peaks above 3% relative abundance were annotated
- Isomer separation: glycan species sharing the same mass (e.g. A2G0M4F / A1G1F(α1,6) / A1G1F(α1,3)) should be reported as separate isomer peaks at distinct RTs within the ±1.75 min window
- MS1 output Excel contains the required columns (glycan name, charge state, theoretical mass, observed m/z, mass error, RT, intensity) and is loadable into the MS2 Peak Confirmation tab

## Limitations

- The isomer peak retention time range is a fixed value (±1.75 min) and cannot be customized
- The app cannot process multiple mzXML files simultaneously; batch processing is a future improvement
- No de novo glycan identification: only glycans already present in the database can be found
- O-linked glycans are outside the app's scope, which is limited to N-linked glycans

## Evidence

- [results] MS1 search tolerances and threshold optimization: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset"
- [results] Isomer detection window and slope criterion: "three isomer peaks were identified at RTs of 13.39, 13.77, and 14.07 min"
- [intro] MS1 tab input requirements: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
- [intro] MS2 Peak Confirmation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Fragment Generator for database expansion: "The databases provided with the app can be expanded through the Fragment Generator functionality which automatically identifies fragmentation patterns for new glycans"
- [abstract] Validation benchmark of manual vs automated identifications: "was successfully validated by identifying all glycan species that were previously identified manually"
- [discussion] Fixed isomer RT window is a limitation: "including the isomer peak retention time range, which is currently a fixed value that meets most use cases"
- [discussion] De novo identification not supported: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database"
- [readme] App license and source location: "This app was developed by Zayla Schaeffer and Ashna Dhingra at AstraZeneca, and is available under MIT license (see License.txt)"
