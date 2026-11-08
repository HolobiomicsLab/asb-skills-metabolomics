---
name: migration-time-index-calibration
description: Use when pre-processing targeted full-scan MSI-CE-MS mzML data where
  migration time drift (~105 s uncorrected) breaks peak alignment, and internal standards
  flank the analytes so MTIs can predict elution times.
when_to_use_negative:
- Input data are not multiplexed MSI-CE-MS runs — the inter-peak spacing confirmation
  and sample position assignment (1-13) assume serial injections within each run
- Sample matrices other than serum (e.g., urine, saliva, stool), for which PeakMeister
  is not yet validated
- Untargeted metabolomics where no targeted mass list or internal-standard reference
  migration times are available
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R software that computes migration time indexes from internal-standard MTs,
    predicts analyte migration times, annotates, integrates, and assigns sample positions
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed (script
    titled 'code'); stats::ksmooth used for EIE smoothing
- name: mzML
  role: Open-source format to which MSI-CE-MS raw data must be converted before PeakMeister
    pre-processing
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:722021edd08b7c049518b3960ededa99af57daf8e882509ede1536e9d40953b6
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# migration-time-index-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates metabolite migration times in multiplexed MSI-CE-MS runs using migration time indexes (MTIs) computed from a panel of internal standards, replacing time-warping or effective-mobility alignment with a computationally simple interpolation between each analyte's flanking standards.

## When to use

Use when pre-processing targeted full-scan MSI-CE-MS mzML data where migration time drift (~105 s uncorrected) breaks peak alignment, and internal standards flank the analytes so MTIs can predict elution times.

## When NOT to use

- Input data are not multiplexed MSI-CE-MS runs — the inter-peak spacing confirmation and sample position assignment (1-13) assume serial injections within each run
- Sample matrices other than serum (e.g., urine, saliva, stool), for which PeakMeister is not yet validated
- Untargeted metabolomics where no targeted mass list or internal-standard reference migration times are available

## Inputs

- mzML-converted MSI-CE-MS raw data files placed in the 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction.window.ppm, and reference migration times
- Reference Mass List sheet listing 11 internal standards with class='Reference' and min.mt.min / max.mt.min cutoffs

## Outputs

- Table of migration time indexes (or relative migration times) used to annotate analytes
- Table of predicted/expected peak migration times
- Table of peak areas
- Plots of each extracted ion electropherogram for annotation QC

## How to apply

Populate the 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction.window.ppm (minimum ~30-35 ppm), and reference migration times of all analytes and internal standards taken from the same run. Mark the 11 internal standards as class 'Reference' in the Reference Mass List sheet and constrain landmark peak candidates with min.mt.min / max.mt.min cutoffs and minimum.peak.width.seconds. Execute the PeakMeister R script, which extracts ion electropherograms from the 'mzML Files' folder (optionally smoothed via stats::ksmooth), computes MTIs from internal-standard migration times, and predicts each analyte's migration time from its two flanking standards, searching within migration.window.seconds. Confirm annotations using the inter-peak spacing check (peak.space.tolerance.percent), since gaps between serially injected segments are typically consistent in MSI-CE-MS. For metabolites lacking two flanking standards, fall back to stable-isotope standards, closely comigrating standards, or relative migration times (RMTs).

## Related tools

- **PeakMeister** (R software that computes migration time indexes from internal-standard MTs, predicts analyte migration times, annotates, integrates, and assigns sample positions) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed (script titled 'code'); stats::ksmooth used for EIE smoothing)
- **mzML** (Open-source format to which MSI-CE-MS raw data must be converted before PeakMeister pre-processing)

## Examples

```
Rscript code  # open the PeakMeister project file and execute the 'code' R script after populating 'Mass List and Parameters.xlsx' and filling 'mzML Files'
```

## Evaluation signals

- Median absolute error between predicted and observed migration times across the 47 serum metabolites is ~2.9 s, far below the ~105 s uncorrected drift (36-fold improvement)
- Median peak annotation accuracy exceeds 99.9%
- Reference internal-standard peaks are the most intense reliable signals in their EIEs and fall within min.mt.min / max.mt.min cutoffs
- Inter-peak spacing between serially injected segments deviates less than peak.space.tolerance.percent from the median spacing
- Mean peak integration bias vs. manual preprocessing by an experienced analyst is -2.1%

## Limitations

- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit are prone to peak misannotation
- extraction.window.ppm must be at least ~30-35 ppm depending on analyte mass and the mass spectrometer used; reference migration times must all come from the same analytical run
- Not yet validated for matrices other than serum (urine, saliva, stool); no user-friendly interface or real-time preprocessing yet available

## Evidence
