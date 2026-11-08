---
name: msi-ce-ms-metabolite-peak-annotation
description: Use when preprocessing targeted full-scan MSI-CE-MS mzML files that require
  peak annotation, peak integration, and sample position assignment despite large
  migration time variability between runs, batches, or months of acquisition.
when_to_use_negative:
- Data are not MSI-CE-MS acquisitions (e.g. conventional single-injection CE-MS, LC-MS,
  or HILIC separations where the equal peak-spacing logic of multiplexed injections
  does not apply).
- Input is already a preprocessed peak-area or feature table rather than raw mzML
  files, since PeakMeister performs its own peak picking, annotation, and integration
  directly on mzML files.
- Non-targeted discovery analyses without a targeted mass list and corresponding internal
  standards, as the method requires a priori masses plus reference peaks for migration
  index computation.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakMeister
  role: Open-source R software that executes the entire MTI-based peak annotation,
    integration, and sample position assignment workflow on MSI-CE-MS mzML files.
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is implemented and executed;
    the 'code' R script in the project file drives preprocessing.
- name: mzML
  role: Open-source file format that all raw data must be converted to before PeakMeister
    preprocessing.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:7631ea61a5549c911da59daa2e9e1a3a79a795ad70cd5ddd41511e37cf87e668
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# msi-ce-ms-metabolite-peak-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated peak annotation and integration for multisegment injection-capillary electrophoresis-mass spectrometry (MSI-CE-MS) targeted full-scan data using migration time indices (MTI) computed from a panel of 11 internal standards, implemented in the open-source R tool PeakMeister.

## When to use

Use when preprocessing targeted full-scan MSI-CE-MS mzML files that require peak annotation, peak integration, and sample position assignment despite large migration time variability between runs, batches, or months of acquisition.

## When NOT to use

- Data are not MSI-CE-MS acquisitions (e.g. conventional single-injection CE-MS, LC-MS, or HILIC separations where the equal peak-spacing logic of multiplexed injections does not apply).
- Input is already a preprocessed peak-area or feature table rather than raw mzML files, since PeakMeister performs its own peak picking, annotation, and integration directly on mzML files.
- Non-targeted discovery analyses without a targeted mass list and corresponding internal standards, as the method requires a priori masses plus reference peaks for migration index computation.

## Inputs

- Targeted full-scan MSI-CE-MS data files converted to mzML format
- Excel template 'Mass List and Parameters.xlsx' with analyte masses, extraction windows, interference declarations, and peak-detection parameters
- Reference mass list of internal standards (class = 'Reference') with migration time cutoffs (min.mt.min, max.mt.min)

## Outputs

- Table of migration indexes / relative migration times used for peak annotation
- Copy of the processing parameters used
- Extracted ion electropherogram plots for visual verification of peak annotation and integration
- Table of peak migration times or expected peak positions
- Table of peak areas (with signals below snr.threshold recorded as '<LOD')

## How to apply

Convert all data files to mzML and place them in an 'mzML Files' folder, then fill the 'Mass List and Parameters.xlsx' template: Sheet 1 lists analyte names, masses (mz), an extraction.window.ppm of at least ~30-35 ppm, interference declarations with interference.comigration.threshold, minimum.peak.width.seconds for noise filtering, migration.window.seconds for the ± search window around the predicted migration time, peak.space.tolerance.percent for the equally-spaced-peak check, and snr.threshold so signals below it are recorded as '<LOD'. Sheet 2 flags internal standards as 'Reference' peaks with min.mt.min and max.mt.min cutoffs; PeakMeister computes a migration time index from 11 internal standards to predict each analyte's migration time, avoiding time warping or effective mobility conversion, then uses the median spacing between peaks to confirm or re-analyze annotations. Results are saved as tables of migration indexes/parameters/peak areas plus extracted ion electropherogram plots for manual verification of annotation and integration fidelity.

## Related tools

- **PeakMeister** (Open-source R software that executes the entire MTI-based peak annotation, integration, and sample position assignment workflow on MSI-CE-MS mzML files.) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is implemented and executed; the 'code' R script in the project file drives preprocessing.)
- **mzML** (Open-source file format that all raw data must be converted to before PeakMeister preprocessing.)

## Examples

```
Rscript code  # open the project file and run the PeakMeister 'code' script with 'Mass List and Parameters.xlsx' and mzML files in the 'mzML Files' folder
```

## Evaluation signals

- Median peak annotation accuracy should exceed 99.9% when compared against manual annotation by an experienced analyst.
- Intermediate precision across batches should yield a median coefficient of variation (CV) around 16.0%.
- Peak integration bias versus manual preprocessing should be small (reported mean bias = -2.1%).
- Quantification of NIST SRM-1950 plasma metabolites should agree with manual values (mean bias = -1.3% for 16 metabolites).
- Extracted ion electropherogram plots must show correctly annotated peaks with no spurious misannotations, especially for late-migrating or low-abundance metabolites.

## Limitations

- Certain late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation.
- Low abundance metabolites with signals near the method detection limit are prone to peak misannotation.
- The required minimum extraction window (~30-35 ppm) is mass-spectrometer dependent and may need tuning per instrument.
- PeakMeister is script-based; a more user-friendly interface is still future work.

## Evidence
