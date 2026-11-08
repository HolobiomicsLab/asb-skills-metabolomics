---
name: migration-index-based-peak-annotation
description: Use when preprocessing targeted full-scan MSI-CE-MS data with large,
  nonlinear migration time fluctuations across batches, where peak alignment and metabolite
  annotation must be automated at scale (e.g., thousands of serum samples).
when_to_use_negative:
- Input data are already preprocessed feature/peak-area tables rather than raw MSI-CE-MS
  mzML files
- Analytes are late-migrating serum metabolites lacking two flanking or a stable-isotope
  internal standard — these are prone to peak misannotation and need manual review
- Sample matrices other than serum (e.g., urine, saliva, stool) — PeakMeister has
  not yet been validated for these matrices
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R software tool implementing MTI-based migration time prediction, peak annotation,
    and integration for targeted full-scan MSI-CE-MS data
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed; smoothing
    uses the stats package 'ksmooth' function
- name: mzML
  role: Open-source file format that all PeakMeister preprocessing is performed on
    directly
- name: MSI-CE-MS
  role: High-throughput separation platform (<4 min/sample via serial injections)
    whose multiplexed datasets the annotation algorithm is designed for
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:6264c166966a2e028ff38474030a00f17d0ab7185e5cdf72db8ceeb6d63cc1b3
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# migration-index-based-peak-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate and integrate metabolite peaks in multiplexed MSI-CE-MS electropherograms by computing migration time indices (MTI) from a panel of 11 internal standards, rather than time-warping or effective-mobility transformations. This is the core algorithm implemented in PeakMeister for high-fidelity automated peak annotation of serum metabolomics data.

## When to use

Use when preprocessing targeted full-scan MSI-CE-MS data with large, nonlinear migration time fluctuations across batches, where peak alignment and metabolite annotation must be automated at scale (e.g., thousands of serum samples).

## When NOT to use

- Input data are already preprocessed feature/peak-area tables rather than raw MSI-CE-MS mzML files
- Analytes are late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard — these are prone to peak misannotation and need manual review
- Sample matrices other than serum (e.g., urine, saliva, stool) — PeakMeister has not yet been validated for these matrices

## Inputs

- Targeted full-scan MSI-CE-MS acquisition files converted to open-source mzML format (placed in the 'mzML Files' folder)
- 'Mass List and Parameters.xlsx' template: Sheet 1 (analyte name, mz, extraction.window.ppm, migration times), Sheet 2 (reference mass list with class, min.mt.min, max.mt.min), Sheet 3 (number.of.injections, lock-mass and smoothing parameters)
- Peak migration times for analytes and internal standards taken from the same run

## Outputs

- Table of migration indexes / relative migration times used to annotate analytes
- Copy of the preprocessing parameters used
- Plots of each extracted ion electropherogram for checking peak annotation and integration
- Table of migration times of peaks or expected peak positions
- Table of peak areas

## How to apply

Convert all MSI-CE-MS acquisition files to open-source mzML and place them in the 'mzML Files' folder, then fill the 'Mass List and Parameters.xlsx' template with analyte m/z values, extraction windows (~30-35 ppm minimum), and reference-peak migration times taken from a single run. PeakMeister computes migration time indices from 11 internal standards (or any reliable high-S/N landmark signals) to predict each analyte's migration time, then searches for peaks within migration.window.seconds of the prediction without dynamic time warping or electrophoretic mobility conversion. It exploits the approximately equal spacing of MSI-CE-MS analytical peaks — governed by peak.space.tolerance.percent — to confirm or adjust annotations, filters noise via minimum.peak.width.seconds, flags signals below snr.threshold as '<LOD', and flags overlapping signals via interference.comigration.threshold. Landmark/reference peaks require reliable and consistently the most intense peak areas within their extracted ion electropherograms, with mass correction applied using lock-mass parameters (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts).

## Related tools

- **PeakMeister** (R software tool implementing MTI-based migration time prediction, peak annotation, and integration for targeted full-scan MSI-CE-MS data) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed; smoothing uses the stats package 'ksmooth' function)
- **mzML** (Open-source file format that all PeakMeister preprocessing is performed on directly)
- **MSI-CE-MS** (High-throughput separation platform (<4 min/sample via serial injections) whose multiplexed datasets the annotation algorithm is designed for)

## Examples

```
Use the project file to open the R script titled "code" and execute the script to begin pre-processing your data
```

## Evaluation signals

- Median peak annotation accuracy >99.9% when annotations are compared against experienced-analyst manual integration
- Mean bias of approximately -2.1% for metabolite peak integration relative to manual preprocessing
- Median intermediate precision CV = 16.0% across samples
- All reported peak migration times for analytes and internal standards are drawn from the same run
- Signal-to-noise below snr.threshold is recorded as '<LOD' rather than as a quantified peak

## Limitations

- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit are prone to peak misannotation
- Extraction window must be tuned per mass spectrometer; ~30-35 ppm is a minimum that depends on analyte mass and instrument
- Reference/landmark peak annotation requires the peak areas to be reliable and always the most intense within the EIE

## Evidence
