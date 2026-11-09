---
name: migration-index-based-migration-time-prediction
description: Use when preprocessing targeted full-scan MSI-CE-MS mzML data where migration
  times vary substantially across runs or batches (e.g. nonlinear electroosmotic flow
  changes) and analyte peaks must be annotated and integrated without manual realignment.
when_to_use_negative:
- Data are already aligned or come from a separation technique with stable migration
  times where simple retention-time matching suffices (the MTI machinery is specific
  to MSI-CE-MS variability).
- The analysis is untargeted feature discovery rather than targeted annotation of
  a known analyte mass list — PeakMeister requires a targeted mass list with known
  m/z values.
- Late-migrating metabolites lack two flanking or a stable-isotope internal standard
  — the article warns these are prone to peak misannotation and should not be trusted
  without extra validation.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R software that computes migration indexes from internal standards to predict
    analyte migration times and drive peak annotation/integration
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister script is executed
- name: MSI-CE-MS
  role: Data acquisition platform producing the multiplexed electropherograms that
    the MTI approach annotates
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:a88337cdadbf193b10c88e1690498100bfbe36855f0b67a659801e33970c5c59
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
  - NIST SRM-1950
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# migration-index-based-migration-time-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict analyte migration times in multiplexed MSI-CE-MS runs using migration time indices (MTI) computed from a panel of internal standards, avoiding computationally intensive dynamic time warping or effective mobility transformations. Implemented in PeakMeister to enable reliable peak annotation across batches despite large migration time fluctuations.

## When to use

Use when preprocessing targeted full-scan MSI-CE-MS mzML data where migration times vary substantially across runs or batches (e.g. nonlinear electroosmotic flow changes) and analyte peaks must be annotated and integrated without manual realignment.

## When NOT to use

- Data are already aligned or come from a separation technique with stable migration times where simple retention-time matching suffices (the MTI machinery is specific to MSI-CE-MS variability).
- The analysis is untargeted feature discovery rather than targeted annotation of a known analyte mass list — PeakMeister requires a targeted mass list with known m/z values.
- Late-migrating metabolites lack two flanking or a stable-isotope internal standard — the article warns these are prone to peak misannotation and should not be trusted without extra validation.

## Inputs

- mzML files of MSI-CE-MS runs placed in a 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, and parameters (extraction.window.ppm, migration.window.seconds, snr.threshold, peak.space.tolerance.percent, min.mt.min/max.mt.min)
- Reference mass list (Sheet 2) designating internal standards as 'Reference' landmark peaks

## Outputs

- Table of migration indexes or relative migration times used for annotation
- Table of peak migration times (or expected peak positions)
- Table of peak areas
- Copy of the processing parameters used
- Extracted ion electropherogram plots per analyte for annotation/integration QC

## How to apply

In the 'Mass List and Parameters.xlsx' template, designate a panel of internal standards (11 used in the article) as 'Reference' class landmarks in Sheet 2 with reliable, always-most-intense peak areas within min.mt.min/max.mt.min cutoffs; PeakMeister computes migration indexes from these landmarks and predicts analyte migration times rather than performing time alignment or mobility conversion. Set migration.window.seconds so analyte peaks are searched at the predicted MT +/- that window, and use peak.space.tolerance.percent to flag peaks whose spacing deviates from the median gap between multiplexed injection peaks, triggering reanalysis. Use extraction.window.ppm of at least ~30-35 ppm to extract each m/z, and mark signals below snr.threshold as '<LOD'. Verify predicted vs. observed migration times by inspecting the extracted ion electropherogram plots and the output tables of migration indexes/relative migration times and peak migration times.

## Related tools

- **PeakMeister** (R software that computes migration indexes from internal standards to predict analyte migration times and drive peak annotation/integration) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister script is executed)
- **MSI-CE-MS** (Data acquisition platform producing the multiplexed electropherograms that the MTI approach annotates)

## Examples

```
Rscript code  # open the PeakMeister R project script with 'Mass List and Parameters.xlsx' filled and mzML files in 'mzML Files'
```

## Evaluation signals

- Peak annotation fidelity: median annotation accuracy >99.9% across runs.
- Intermediate precision: median CV of 16.0% across QC samples.
- Peak integration agreement vs. manual processing: mean bias of -2.1%.
- External quantification check vs. NIST SRM-1950: mean bias of -1.3% across the 16 shared plasma metabolites, with per-metabolite bias within -28% (alanine) to +25% (cystine).
- Per-run: extracted ion electropherogram plots show peaks annotated at predicted positions with spacing consistent within peak.space.tolerance.percent.

## Limitations

- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation.
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation.
- In one ENANI-2019 data file, low ornithine-15N internal standard signal caused 73 peaks to be misannotated, requiring that outlier file to be excluded.
- Extraction window of at least ~30-35 ppm is required and is dependent on the mass spectrometer used during acquisition.

## Evidence
