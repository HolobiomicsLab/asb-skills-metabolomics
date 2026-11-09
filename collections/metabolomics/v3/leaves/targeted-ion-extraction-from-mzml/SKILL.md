---
name: targeted-ion-extraction-from-mzml
description: Use when input is raw full-scan MSI-CE-MS data in mzML format plus a
  targeted mass list (analyte names, m/z values, reference migration times) and ion
  electropherograms must be extracted for peak annotation.
when_to_use_negative:
- Input is not targeted full-scan MSI-CE-MS data (e.g. untargeted feature tables or
  LC-MS/MS data) — PeakMeister was designed for multiplexed MSI-CE-MS datasets with
  internal-standard landmarks
- Data is still in vendor raw format and cannot be converted to mzML, since all PeakMeister
  preprocessing is performed directly on mzML files
- Sample matrices other than serum (urine, saliva, stool), for which PeakMeister is
  not yet validated
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R-based software that performs targeted ion extraction and preprocessing of
    MSI-CE-MS mzML files using migration indexes
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed; ksmooth
    (stats package) used for EIE smoothing
  repo: github.com/PBM-Group/PeakMeister
- name: mzML
  role: Open-source file format that all PeakMeister input data must be converted
    to before extraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:19fd18d32e5894bc25d05fa6750c0a43f58a5d13ff89e5d2ad3b042fba984418
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Targeted ion extraction from mzML

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts ion electropherograms for a targeted metabolite list from MSI-CE-MS mzML files using PeakMeister, an R-based tool, with per-analyte ppm extraction windows and smoothing for downstream peak annotation and integration.

## When to use

Use when input is raw full-scan MSI-CE-MS data in mzML format plus a targeted mass list (analyte names, m/z values, reference migration times) and ion electropherograms must be extracted for peak annotation.

## When NOT to use

- Input is not targeted full-scan MSI-CE-MS data (e.g. untargeted feature tables or LC-MS/MS data) — PeakMeister was designed for multiplexed MSI-CE-MS datasets with internal-standard landmarks
- Data is still in vendor raw format and cannot be converted to mzML, since all PeakMeister preprocessing is performed directly on mzML files
- Sample matrices other than serum (urine, saliva, stool), for which PeakMeister is not yet validated

## Inputs

- Full-scan MSI-CE-MS raw data converted to mzML files (placed in the 'mzML Files' folder)
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction.window.ppm, and reference migration times from a single run
- Reference Mass List sheet listing internal standards (class = 'Reference') with min.mt.min / max.mt.min cutoffs
- Parameters sheet: number.of.injections, lock-mass settings (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts), apply.mass.correction, apply.smoothing

## Outputs

- Extracted ion electropherograms (EIE plots per analyte for checking annotation and integration)
- Table of migration indexes or relative migration times used to annotate analytes
- Table of migration times of peaks or expected peak positions
- Table of peak areas
- Copy of the parameters used to process the data

## How to apply

Convert all raw data to mzML files and place them in the 'mzML Files' folder required by PeakMeister; populate the 'Mass List and Parameters.xlsx' template with unique analyte names, m/z values, and an extraction.window.ppm per analyte (a minimum window of ~30-35 ppm is required depending on analyte mass and the mass spectrometer used). Optionally enable smoothing via the 'ksmooth' function from the R stats package by setting apply.smoothing = 'Yes', smoothing.kernal, and smoothing.strength. Set migration.window.seconds so PeakMeister searches for analyte peaks at the expected migration time +/- this window, and set reference compounds with class = 'Reference' plus min.mt.min / max.mt.min cutoffs so only peaks within thresholds are considered as reference standard peaks. Execute the R script titled 'code' from the project file to run extraction; rationale is that PeakMeister operates directly on mzML files without time-warping or electrophoretic mobility conversion, using migration indexes from internal standards instead.

## Related tools

- **PeakMeister** (R-based software that performs targeted ion extraction and preprocessing of MSI-CE-MS mzML files using migration indexes) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed; ksmooth (stats package) used for EIE smoothing) — github.com/PBM-Group/PeakMeister
- **mzML** (Open-source file format that all PeakMeister input data must be converted to before extraction)

## Examples

```
Rscript code  # open the PeakMeister project file and execute the 'code' R script after populating 'Mass List and Parameters.xlsx' and filling the 'mzML Files' folder
```

## Evaluation signals

- All mzML files are present in the 'mzML Files' folder and every analyte name in the Mass List sheet is unique
- Every extraction.window.ppm is at least ~30-35 ppm, depending on analyte mass, so ion electropherograms are fully extracted
- Reference migration times for analytes and internal standards are all taken from the same analytical run
- Exported EIE plots show correct peak annotation and integration at the predicted migration time +/- migration.window.seconds
- Output tables (migration index table, peak migration-time table, peak-area table) and the parameter copy are all saved by the run

## Limitations

- Minimum extraction window of ~30-35 ppm depends on analyte mass and the mass spectrometer used during acquisition, so windows may need per-analyte tuning
- Lock-mass correction is only applied when ref.mass.counts (minimum lock-mass peak height) is met and apply.mass.correction is set to 'Yes'
- PeakMeister is script-based (no user-friendly interface yet) and has only been demonstrated for serum metabolomics, not other matrices
- Peaks with signal below snr.threshold are recorded as '<LOD' rather than quantified

## Evidence
