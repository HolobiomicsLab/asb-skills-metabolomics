---
name: stable-isotope-dilution-calibration
description: Use when targeted LC-SRM or HRMS chromatograms contain stable-isotope-labeled
  internal standards for each analyte and the goal is absolute concentration (µM,
  nmol/g, µmol/mmol creatinine) rather than relative peak-area comparison.
when_to_use_negative:
- Input is already a feature table from untargeted profiling (XCMS/MZmine/MS-DIAL)
  with no stable-isotope-labeled internal standards available for each analyte — peak-area
  ratio calibration cannot be performed.
- The experiment requires relative abundance comparisons across conditions rather
  than absolute concentrations; unlabeled external calibration alone would suffice.
- GC-MS or ion-mobility small-molecule data are being analyzed; Skyline support for
  these is explicitly still ongoing, so isotope-dilution calibration in Skyline is
  not yet the right tool.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Skyline
  role: Software in which the transition list, heavy-labeled internal standards, and
    calibration curve are combined to compute analyte concentrations from peak-area
    ratios.
  repo: github.com/ProteoWizard/pwiz
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:508bc82d8839cc8103b03e40ae6dff314c6ce8180f7ae5db9bdb71856d42b061
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# stable-isotope-dilution-calibration

## Summary

Calibrates LC-SRM/HRMS peak-area ratios against a ten-point 0.01–25 µM external standard curve spiked with heavy-labeled internal standards, exporting analyte concentrations as CSV from a Skyline document.

## When to use

Use when targeted LC-SRM or HRMS chromatograms contain stable-isotope-labeled internal standards for each analyte and the goal is absolute concentration (µM, nmol/g, µmol/mmol creatinine) rather than relative peak-area comparison.

## When NOT to use

- Input is already a feature table from untargeted profiling (XCMS/MZmine/MS-DIAL) with no stable-isotope-labeled internal standards available for each analyte — peak-area ratio calibration cannot be performed.
- The experiment requires relative abundance comparisons across conditions rather than absolute concentrations; unlabeled external calibration alone would suffice.
- GC-MS or ion-mobility small-molecule data are being analyzed; Skyline support for these is explicitly still ongoing, so isotope-dilution calibration in Skyline is not yet the right tool.

## Inputs

- Skyline transition list containing endogenous targets plus their heavy-labeled internal standards (e.g. IROA MSMLS for purine/pyrimidine panel)
- Raw LC-SRM vendor files from instruments such as SCIEX 5500 or Waters Xevo TQ-S with RT-scheduled methods
- Ten-point calibration curve series spanning 0.01–25 µM acquired in the same run batch
- Blank-matrix or surrogate-matrix calibration samples matched to the biological matrix (e.g. mouse urine, human brain tissue homogenate)

## Outputs

- Per-analyte peak areas, analyte/internal-standard peak-area ratios, and calculated concentrations exported as CSV (e.g. polyamine_concentrations.csv)
- Calibration-curve fit parameters and peak-integration results reviewed in the Skyline document
- Quantitative table verifying that all named analytes (e.g. 15 polyamine-pathway metabolites such as GABA, Arg, Orn, SAM, SAH, N1-/N8-AcSpd, MTA) have rows with concentrations in µM, nmol/g, or µmol/mmol creatinine

## How to apply

Import the transition list into Skyline with each target and its heavy-labeled internal standard, then integrate extracted ion chromatograms for both endogenous and labeled transitions. Prepare a ten-point 0.01–25 µM calibration curve in the same matrix or surrogate matrix and acquire it in the same run batch. In Skyline, calibrate quantification against the curve fit using peak-area ratios (analyte/internal-standard) so that injection-to-injection variability is absorbed by the labeled standard. For the polyamine assay, apply the RT-scheduled 1-minute window method exported from the Acquity UPLC/Xevo TQ-S PFHA ion-pairing run; for purine/pyrimidine panels, keep the SCIEX 5500 polarity-switching transitions in a single document. Export per-analyte peak areas, analyte/internal-standard ratios, and calculated concentrations (e.g. polyamine_concentrations.csv) and verify the curve is monotonic and all transitions are present.

## Related tools

- **Skyline** (Software in which the transition list, heavy-labeled internal standards, and calibration curve are combined to compute analyte concentrations from peak-area ratios.) — github.com/ProteoWizard/pwiz

## Examples

```
In Skyline: File > Import > Transition List (targets + heavy-labeled standards), then Small Molecules > Calibrate against the ten-point 0.01–25 µM curve, then File > Export > CSV of peak areas, ratios, and concentrations.
```

## Evaluation signals

- The exported CSV contains rows for all 15 named polyamine-pathway analytes with non-missing concentration values.
- Calibration-curve fit is monotonic across the ten-point 0.01–25 µM range and passes Skyline's standard QC.
- Peak integration shows no missing transitions for either endogenous or heavy-labeled standards.
- Concentrations are reported in the expected units for the matrix (µM for serum/calibrators, nmol/g for brain tissue, µmol/mmol creatinine for urine), e.g. mouse urine Arg 4.36 and Orn 3.63 µmol/mmol creatinine.
- Skyline peak-picking yields integrated chromatograms for both positive and negative polarity-switching transitions in the purine/pyrimidine panel (cycle time ~1.3 s).

## Limitations

- Skyline may lag behind commercial software for the latest-generation or vendor-specific workflows until support is implemented.
- Fragmentation library workflows for small molecules (Metlin, MassBank, HMDB, NIST) are not yet implemented, so library-assisted transition confirmation must come from elsewhere.
- No single tool is best for all workflows — GC-MS and ion-mobility small molecule support is still ongoing, so calibration in Skyline is not universally applicable.

## Evidence

- [results] Calibration curve workflow: [evidence span withheld — anchor status: not_found]
