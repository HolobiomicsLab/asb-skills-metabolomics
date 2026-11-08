---
name: cross-matrix-metabolomics-method-transfer
description: Decision support for extending an MTI/internal-standard-calibrated MSI-CE-MS
  annotation pipeline beyond its validated serum matrix to urine, saliva, stool, or
  serial-injection isocratic LC-MS data.
when_to_use_negative:
- Data are already from serum MSI-CE-MS runs — PeakMeister's serum validation (median
  annotation accuracy >99.9%, median CV = 16.0%) already covers this case directly
- Untargeted metabolomic feature finding is the goal — PeakMeister requires a targeted
  mass list and is designed for targeted full-scan MSI-CE-MS pre-processing
- Late-migrating analytes lacking two flanking or stable-isotope internal standards,
  or low-abundance signals near the detection limit, dominate the panel — these were
  prone to peak misannotation even in serum and need special handling before matrix
  transfer
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: PeakMeister
  role: Open-source R tool that computes migration indexes from 11 reference internal
    standards to predict analyte migration times and automate peak annotation, integration,
    and sample position assignment on mzML files
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and executed ('code'
    R script) for pre-processing MSI-CE-MS data
- name: MSI-CE-MS
  role: High-throughput separation platform (<4 min/sample) whose multiplexed serial-injection
    data the transfer evaluation targets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:5897baa6de6626b9a23b2c1b730af6dd0b4c297a5ecd90863fcf8ae5b054013c
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-matrix-metabolomics-method-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates whether the MTI-based PeakMeister preprocessing workflow, validated for serum MSI-CE-MS (median annotation accuracy >99.9%), can be transferred to other sample matrices (urine, saliva, stool) by benchmarking annotation accuracy and peak-area MAE against ground truth.

## When to use

Decision support for extending an MTI/internal-standard-calibrated MSI-CE-MS annotation pipeline beyond its validated serum matrix to urine, saliva, stool, or serial-injection isocratic LC-MS data.

## When NOT to use

- Data are already from serum MSI-CE-MS runs — PeakMeister's serum validation (median annotation accuracy >99.9%, median CV = 16.0%) already covers this case directly
- Untargeted metabolomic feature finding is the goal — PeakMeister requires a targeted mass list and is designed for targeted full-scan MSI-CE-MS pre-processing
- Late-migrating analytes lacking two flanking or stable-isotope internal standards, or low-abundance signals near the detection limit, dominate the panel — these were prone to peak misannotation even in serum and need special handling before matrix transfer

## Inputs

- mzML files of MSI-CE-MS (or serial-injection isocratic LC-MS) extracted-ion electropherograms from the non-serum matrix
- Ground-truth annotation and peak-area table for the analyte panel
- 'Mass List and Parameters.xlsx' template (Sheets 1-3: mass list, reference mass list, parameters)
- Internal-standard reference panel (e.g., 11 reference standards) with migration-time cutoffs

## Outputs

- Migration-index table used for analyte annotation
- Migration-time table (predicted and observed peak positions)
- Peak-area table
- Extracted-ion electropherogram plots for checking annotation and integration
- annotation_accuracy_mae.csv with annotation accuracy and MAE vs. ground truth

## How to apply

Simulate or acquire extracted-ion electropherograms for the target matrix with a targeted analyte panel flanked by an 11-internal-standard reference set, keeping the matrix-specific interferences, noise, and migration-time variability realistic, and export them as mzML files into the 'mzML Files' folder. Fill Sheet 1 of the 'Mass List and Parameters.xlsx' template with unique analyte names, m/z values, extraction.window.ppm (>= ~30-35 ppm minimum), minimum.peak.width.seconds, migration.window.seconds, peak.space.tolerance.percent, snr.threshold, and ksmooth kernel/bandwidth; Sheet 2 with reference standards (class='Reference', min.mt.min/max.mt.min cutoffs); Sheet 3 with number.of.injections and lock-mass parameters (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts). Run the PeakMeister R script so it computes migration indexes from the reference standards, predicts analyte migration times, annotates and integrates peaks, and emits the migration-index, migration-time, and peak-area tables plus EIE plots. Finally compare predicted peak positions, annotations, and integrated areas against ground truth, computing annotation accuracy and mean absolute error (MAE), and judge transfer success against the paper's serum benchmark of median annotation accuracy >99.9% and mean integration bias = -2.1%.

## Related tools

- **PeakMeister** (Open-source R tool that computes migration indexes from 11 reference internal standards to predict analyte migration times and automate peak annotation, integration, and sample position assignment on mzML files) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and executed ('code' R script) for pre-processing MSI-CE-MS data)
- **MSI-CE-MS** (High-throughput separation platform (<4 min/sample) whose multiplexed serial-injection data the transfer evaluation targets)

## Examples

```
Rscript code  # open the PeakMeister project file and execute the 'code' R script after placing mzML files in 'mzML Files' and filling 'Mass List and Parameters.xlsx'
```

## Evaluation signals

- Reported annotation accuracy and MAE are quantified against a ground-truth annotation/peak-area table rather than asserted qualitatively
- Median annotation accuracy in the new matrix is benchmarked against the paper's serum reference performance of median accuracy >99.9%
- Peak-area integration bias falls near the serum benchmark (mean bias = -2.1%) and intermediate precision near median CV = 16.0%
- All analyte peak migration times used for initial calibration are taken from the same run, including the reference-standard values in Sheet 2
- EIE plots confirm correct peak annotation and integration for every analyte, with interferences flagged via interference/interference.comigration.threshold settings

## Limitations

- PeakMeister has not been validated for matrices other than serum (urine, saliva, stool) — transfer is an open research task, not an established capability
- Certain late-migrating metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit were prone to peak misannotation
- No user-friendly software interface is available yet; no real-time (as opposed to retrospective) pre-processing output capability exists
- A minimum mass extraction window of ~30-35 ppm is required, and the optimal window depends on the mass spectrometer used

## Evidence

- [discussion] high throughput MSI-CE-MS metabolomic analyses in different sample matrices (e.g., urine, saliva, stool): "as required for high throughput MSI-CE-MS metabolomic analyses in different sample matrices (e.g., urine, saliva, stool)"
- [abstract] excellent peak annotation fidelity (median accuracy >99.9%): "excellent peak annotation fidelity (median accuracy >99.9%)"
- [readme] We found that a minimum mass window of ~30-35 ppm is required: "We found that a minimum mass window of ~30-35 ppm is required depending on the mass of the analyte"
- [readme] Peaks that elute before this threshold will not be considered as possible reference standard peaks.: "min.mt.min - Minimum migration time cutoff. Peaks that elute before this threshold will not be considered as possible reference standard peaks."
- [abstract] consistent metabolite peak integration (mean bias = -2.1%): "consistent metabolite peak integration (mean bias = -2.1%)"
