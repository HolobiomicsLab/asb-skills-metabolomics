---
name: msi-ce-ms-data-preprocessing
description: Use when preprocessing large batches of targeted full-scan MSI-CE-MS
  data (mzML files) where large migration time fluctuations across batches make manual
  vendor-software peak annotation/integration prohibitively slow, and accurate peak
  annotation, integration, and sample position assignment are.
when_to_use_negative:
- Data are not from MSI-CE-MS multiplexed separations — the tool relies on consistent
  peak spacing between serial injections and CE-MS migration time behavior; HILIC-based
  LC-MS data or already-processed feature tables are out of scope
- Sample matrices other than serum — PeakMeister is not yet validated for urine, saliva,
  or stool
- A user-friendly GUI is required — only a script-based R workflow with an Excel parameter
  sheet is available
- Untargeted metabolomic feature detection is needed — the workflow requires a targeted
  mass list with known m/z values
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: 'Core tool: automated migration-time prediction via migration time indices,
    peak annotation, integration, and sample position assignment of targeted full-scan
    MSI-CE-MS mzML data'
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed (via
    the project file and the 'code' script)
- name: mzML
  role: Required open-source input file format for all preprocessing
- name: MSI-CE-MS
  role: High-throughput separation platform (<4 min/sample) whose multiplexed data
    the preprocessing targets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:87684e0601972bacf0318a3c6008ab6da64a43b5d7ffe4cbd57c5310e6741ec0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msi-ce-ms-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated targeted preprocessing of multisegment injection-capillary electrophoresis-mass spectrometry (MSI-CE-MS) full-scan metabolomic data using PeakMeister, an R-based tool that predicts analyte migration times via migration time indices (MTI) from 11 internal standards instead of dynamic time warping or electrophoretic mobility transformations, reducing preprocessing of >5,000-sample studies from ~125 days manual work to ~4 days (>30-fold speedup).

## When to use

Use when preprocessing large batches of targeted full-scan MSI-CE-MS data (mzML files) where large migration time fluctuations across batches make manual vendor-software peak annotation/integration prohibitively slow, and accurate peak annotation, integration, and sample position assignment are needed.

## When NOT to use

- Data are not from MSI-CE-MS multiplexed separations — the tool relies on consistent peak spacing between serial injections and CE-MS migration time behavior; HILIC-based LC-MS data or already-processed feature tables are out of scope
- Sample matrices other than serum — PeakMeister is not yet validated for urine, saliva, or stool
- A user-friendly GUI is required — only a script-based R workflow with an Excel parameter sheet is available
- Untargeted metabolomic feature detection is needed — the workflow requires a targeted mass list with known m/z values

## Inputs

- MSI-CE-MS full-scan acquisition files converted to open-source mzML format (placed in 'mzML Files' folder)
- Targeted mass list with analyte names and m/z values ('Mass List and Parameters.xlsx', Sheet 1)
- Preprocessing parameters: extraction.window.ppm, migration.window.seconds, minimum.peak.width.seconds, peak.space.tolerance.percent, snr.threshold, smoothing.kernal, smoothing.strength
- Reference/internal standard mass list with MTs from a single run (Sheet 2), including class='Reference', min.mt.min, max.mt.min
- Lock-mass/parameters sheet: number.of.injections, ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts, apply.mass.correction, apply.smoothing

## Outputs

- Table of migration indexes or relative migration times used for analyte annotation
- Copy of the parameters used to process the data
- Plots of each extracted ion electropherogram for checking peak annotation and integration
- Table of peak migration times (or expected peak positions)
- Table of peak areas (with low-SNR peaks recorded as '<LOD')

## How to apply

Convert all MSI-CE-MS acquisition files (serum/plasma samples, pooled QC, calibrants, blanks) to open-source mzML format and place them in the 'mzML Files' folder required by PeakMeister. Fill the 'Mass List and Parameters.xlsx' template: Sheet 1 holds analyte names, m/z values, and per-analyte parameters such as extraction.window.ppm (~30-35 ppm minimum depending on mass spectrometer), migration.window.seconds (peak search window around predicted MT), minimum.peak.width.seconds (noise filter for peak width), peak.space.tolerance.percent (tolerance on expected equal spacing between multiplexed peaks), snr.threshold (signals below threshold are recorded as '<LOD'), and interference.comigration.threshold for flagged interferences. Sheet 2 designates reference/landmark peaks (typically internal standards, e.g., a panel of 11) used to compute migration time indices, with min.mt.min/max.mt.min cutoffs; Sheet 3 sets lock-mass correction parameters (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts). Open the R script titled 'code' via the project file and execute; PeakMeister predicts analyte migration times from MTIs of the internal standards (avoiding time-warping or effective mobility conversion), confirms annotations using the consistent peak spacing of MSI-CE-MS multiplexed injections, performs peak picking, integration, and sample position assignment, and saves results. Run PeakMeister on the same sample set manually integrated by an experienced analyst to validate by comparing peak areas (mean bias), annotation accuracy, and intermediate precision (CV).

## Related tools

- **PeakMeister** (Core tool: automated migration-time prediction via migration time indices, peak annotation, integration, and sample position assignment of targeted full-scan MSI-CE-MS mzML data) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed (via the project file and the 'code' script))
- **mzML** (Required open-source input file format for all preprocessing)
- **MSI-CE-MS** (High-throughput separation platform (<4 min/sample) whose multiplexed data the preprocessing targets)

## Examples

```
Rscript code  # open the PeakMeister project file in R and execute the 'code' script with mzML files in 'mzML Files/' and parameters in 'Mass List and Parameters.xlsx'
```

## Evaluation signals

- Median peak annotation accuracy >99.9% across the processed run set
- Intermediate precision: median CV = 16.0% for metabolite quantification
- Mutual agreement with manual experienced-analyst preprocessing: mean bias = -2.1% for metabolite peak integration
- Benchmark vs NIST SRM-1950: mean bias = -1.3% for quantifying 16 plasma metabolites (468 NIST SRM-1950 plasma samples)
- Speedup: automated preprocessing of ~5,420 ENANI-2019 samples completed within ~4 days on a desktop Core i7-9700 vs ~125 days manual, i.e., >30-fold reduction

## Limitations

- Late migrating metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation
- Not yet validated for sample matrices other than serum (e.g., urine, saliva, stool)
- No user-friendly GUI; 'real-time' preprocessing output for review not yet implemented (currently retrospective)
- extraction.window.ppm depends on the mass spectrometer used during acquisition; ~30-35 ppm was sufficient in this study

## Evidence

  * Provide a targeted mass list and the corresponding parameters using the provided "Mass List"
