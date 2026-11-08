---
name: r-script-workflow-execution
description: Use when a script-based R tool must be executed on a batch of raw data
  files — e.g. converting vendor MSI-CE-MS files to mzML, filling an Excel parameter
  template, and running the project-file R script to automate preprocessing at scale.
when_to_use_negative:
- 'Data are untargeted / non-multiplexed: PeakMeister requires a targeted mass list
  and relies on consistent peak spacing in MSI-CE-MS runs, so it is the wrong choice
  for feature-discovery workflows or non-MSI-CE-MS separations'
- 'The sample matrix is not serum-like: PeakMeister has not yet been validated for
  urine, saliva, or stool matrices'
- Only one file needs inspection — manual vendor-software integration may be simpler
  than configuring the Excel template and running the script
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R-script tool executed via its project file to automate targeted MSI-CE-MS
    preprocessing (peak annotation, integration, sample position assignment)
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister script runs; ksmooth from
    the stats package is used for EIE smoothing
- name: mzML
  role: Open-source file format to which all acquisition files must be converted before
    script execution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:3ba2aefafb4368a21df0da400f8ee2ac9bdef6dc6d3e8cb133aa5355c47d75df
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-script-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes a script-based R analysis tool (PeakMeister) end-to-end by preparing its required folder structure and Excel parameter template, running the 'code' R script via the project file, and recording wall-clock processing time. This skill captures how to operationalize a script-only R tool that requires little to no coding.

## When to use

Use when a script-based R tool must be executed on a batch of raw data files — e.g. converting vendor MSI-CE-MS files to mzML, filling an Excel parameter template, and running the project-file R script to automate preprocessing at scale.

## When NOT to use

- Data are untargeted / non-multiplexed: PeakMeister requires a targeted mass list and relies on consistent peak spacing in MSI-CE-MS runs, so it is the wrong choice for feature-discovery workflows or non-MSI-CE-MS separations
- The sample matrix is not serum-like: PeakMeister has not yet been validated for urine, saliva, or stool matrices
- Only one file needs inspection — manual vendor-software integration may be simpler than configuring the Excel template and running the script

## Inputs

- Raw MSI-CE-MS acquisition files converted to mzML format (placed in the 'mzML Files' folder)
- 'Mass List and Parameters.xlsx' template with targeted analyte mass list (name, mz, extraction.window.ppm)
- Sheet 2 reference/internal-standard list with migration-time cutoffs (min.mt.min, max.mt.min)
- Sheet 3 run parameters (number.of.injections, ref.mass.one/two, ref.mass.window.ppm, ref.mass.counts, apply.mass.correction, apply.smoothing)
- PeakMeister project file containing the R script titled 'code'

## Outputs

- Table of migration indexes / relative migration times used for analyte annotation
- Copy of the processing parameters used
- Plots of each extracted ion electropherogram for checking peak annotation and integration
- Table of peak migration times (or expected peak positions)
- Table of peak areas
- Wall-clock processing time on the reference machine for speedup measurement

## How to apply

First, convert all acquisition files (e.g. 5,000 ENANI-2019 serum samples, 420 QC samples, NIST SRM-1950 plasma, calibrants, blanks) to open-source mzML and place them in the 'mzML Files' folder that the tool expects. Second, complete the 'Mass List and Parameters.xlsx' template: Sheet 1 holds the targeted mass list (unique names, mz values, extraction.window.ppm ~30–35 ppm, migration.window.seconds, snr.threshold, minimum.peak.width.seconds, peak.space.tolerance.percent), Sheet 2 lists reference/internal-standard landmark peaks with min.mt.min/max.mt.min cutoffs, and Sheet 3 holds run-level parameters (number.of.injections, lock-mass settings, apply.mass.correction, apply.smoothing). Third, open the R script titled 'code' via the provided project file and execute it; little to no coding knowledge is required since parameters are set in the Excel sheet. Finally, record the wall-clock time on a defined reference machine (e.g. desktop Core i7-9700) so throughput can be compared against a manual baseline — the paper's benchmark was ~4 days automated vs ~125 days manual, a >30-fold speedup.

## Related tools

- **PeakMeister** (R-script tool executed via its project file to automate targeted MSI-CE-MS preprocessing (peak annotation, integration, sample position assignment)) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister script runs; ksmooth from the stats package is used for EIE smoothing)
- **mzML** (Open-source file format to which all acquisition files must be converted before script execution)

## Examples

```
Rscript code  # opened via the PeakMeister project file, with mzML files in 'mzML Files/' and 'Mass List and Parameters.xlsx' completed
```

## Evaluation signals

- All mzML files are present in the 'mzML Files' folder and the 'Mass List and Parameters.xlsx' template is complete before execution (no missing mz values or parameters)
- Script runs to completion from the project file and writes all five result outputs (MTI table, parameter copy, EIE plots, migration-time table, peak-area table)
- Spot-check EIE plots confirm proper peak annotation and integration; anomalous spacing triggers reanalysis via peak.space.tolerance.percent
- Intermediate precision is acceptable (median CV ≈ 16.0%) and mean bias vs manual integration ≈ -2.1% (≈ -1.3% for 16 NIST SRM-1950 metabolites), with median annotation accuracy >99.9%
- Wall-clock time on the reference Core i7-9700 is on the order of days (≈4 days for the full ENANI-2019 set), reproducing the >30-fold speedup vs ~125 days manual work

## Limitations

- No user-friendly interface yet: the tool is script-based and output is produced retrospectively; a 'real-time' preprocessing output capability and a GUI are future work
- Certain late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard, and low-abundance metabolites near the detection limit, are prone to peak misannotation
- Minimum extraction window of ~30–35 ppm is required and may depend on the mass spectrometer; peak-space assumptions (approximate equal spacing) break if runs deviate from the MSI-CE-MS multiplexed design
- Not validated for sample matrices other than serum (e.g. urine, saliva, stool)

## Evidence
