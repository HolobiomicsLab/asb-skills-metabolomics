---
name: mzmine-workflow-configuration
description: Use when building a sample-centric LC-HRMS2 feature detection pipeline
  on UHPLC-MS2 DDA data (mzML/mzXML inputs) that must yield a per-sample feature table
  and MS2 .mgf export for FBMN, ISDB, or KG integration.
when_to_use_negative:
- When the input is already an aligned N x M feature quantification table — the ENPKG
  workflow's motivation is precisely to avoid retention-time-based feature alignment
  across batches.
- When processing data other than LC-HRMS2 DDA metabolomics (e.g., proteomics LC-MS/MS
  or GC-MS EI data), since the ADAP/wavelet parameters and 12 ppm tolerance were tuned
  for this instrumental setup.
- When a downstream tool requires a merged, batch-aligned dataset rather than per-sample
  MZmine output.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Performs the feature detection and export steps of the ENPKG workflow (Figure
    2B, step 1) — mass detection, chromatogram building, deconvolution, isotope grouping,
    feature filtering, and .mgf export.
  repo: https://github.com/enpkg/enpkg_workflow
- name: ProteoWizard
  role: Used via MSConvert as the entry point of the workflow to convert Thermo RAW
    LC-MS files to mzXML format before MZmine processing.
- name: matchms
  role: Downstream consumer of the exported .mgf MS2 spectra and feature quantification
    table for ISDB-based structural annotation after MZmine export.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:6aa4f1c0aff347744d5e07d7e51aa7e2d411ed2198385ece745119b042bb65a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mzmine-workflow-configuration

## Summary

Configure the MZmine stage of the ENPKG pipeline: convert Thermo RAW to mzXML, centroid mass detection, ADAP chromatogram building and wavelet deconvolution, isotope grouping, MS2/RT feature filtering, and export of the feature quantification table plus per-feature MS2 .mgf spectra for downstream annotation and molecular networking.

## When to use

Use when building a sample-centric LC-HRMS2 feature detection pipeline on UHPLC-MS2 DDA data (mzML/mzXML inputs) that must yield a per-sample feature table and MS2 .mgf export for FBMN, ISDB, or KG integration.

## When NOT to use

- When the input is already an aligned N x M feature quantification table — the ENPKG workflow's motivation is precisely to avoid retention-time-based feature alignment across batches.
- When processing data other than LC-HRMS2 DDA metabolomics (e.g., proteomics LC-MS/MS or GC-MS EI data), since the ADAP/wavelet parameters and 12 ppm tolerance were tuned for this instrumental setup.
- When a downstream tool requires a merged, batch-aligned dataset rather than per-sample MZmine output.

## Inputs

- Thermo RAW LC-MS files (UHPLC-HRMS2 DDA, positive and/or negative ionization modes)
- Converted mzXML / mzML files after ProteoWizard MSConvert
- Per-sample minimal metadata file containing the sample's originating taxon

## Outputs

- Feature quantification table per sample with m/z, retention time, and area/intensity columns for each deconvoluted feature
- Per-feature MS2 spectra exported as .mgf files
- Raw LC-HRMS2 data in .mzML format (uploaded to MassIVE)
- Filtered feature lists containing only features with an associated MS2 scan and RT in 0.5-8.0 min

## How to apply

First convert Thermo RAW files to mzXML with MSConvert (ProteoWizard) as MZmine input. Configure MZmine per the ENPKG parameters: centroid mass detector with noise level 1.0E4 for MS level 1 and 0 for MS level 2; ADAP chromatogram builder with minimum group size 5 scans, minimum group intensity 1.0E4, minimum highest intensity 5.0E5, and 12 ppm m/z tolerance; wavelets (ADAP) deconvolution using an intensity-window S/N estimator with S/N 10, minimum feature height 5.0E5, coefficient area threshold 50, peak duration 0.02-0.5 min, and RT wavelet range 0.01-0.03 min. Then run the isotope peaks grouper (12 ppm m/z tolerance, 0.01 min absolute RT tolerance, maximum charge 2, most intense isotope as representative). Filter each feature list with the feature filtering module to keep only features carrying an associated MS2 scan and a retention time between 0.5 and 8.0 min, since the workflow processes each sample individually rather than aligning across batches. Finally export the deconvoluted feature quantification table (m/z, RT, area/intensity per feature) and per-feature MS2 spectra with the 'Export to GNPS' module so the .mzML and .mgf files can feed downstream matchms-based ISDB annotation, SIRIUS/CSI:FingerID, and molecular networking steps.

## Related tools

- **MZmine** (Performs the feature detection and export steps of the ENPKG workflow (Figure 2B, step 1) — mass detection, chromatogram building, deconvolution, isotope grouping, feature filtering, and .mgf export.) — https://github.com/enpkg/enpkg_workflow
- **ProteoWizard** (Used via MSConvert as the entry point of the workflow to convert Thermo RAW LC-MS files to mzXML format before MZmine processing.)
- **matchms** (Downstream consumer of the exported .mgf MS2 spectra and feature quantification table for ISDB-based structural annotation after MZmine export.)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && uv sync && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Verify that the feature quantification table exists with the expected columns (m/z, RT, area/intensity per feature).
- Check that the exported .mgf files contain MS2 spectra for each retained feature — after filtering, every kept feature must have an associated MS2 scan.
- Confirm that no retained feature has a retention time outside the 0.5-8.0 min window.
- Spot-check that MS2 spectra export includes per-feature fragmentation spectra consistent with the raw .mzML files, matching the reported ~788,623 features in PI and ~364,967 in NI for the full 1,600-extract data set before per-sample filtering.
- Confirm that the .mzML and .mgf output files are consumable by the downstream matchms/ISDB and SIRIUS/CSI:FingerID steps without format errors.

## Limitations

- The 12 ppm m/z tolerance, 0.5-8.0 min RT filter, and intensity thresholds (5.0E5 minimum feature height, 1.0E4 noise) are specific to the ENPKG UHPLC-HRMS2 setup and may need re-tuning for other chromatographic or mass-spectrometric configurations.
- Feature detection is done per sample; classical retention-time-based alignment is intentionally absent, which hinders direct comparison of samples processed with different MZmine parameters.
- PI-mode LC-MS2 acquisition parameters for the 1,600-plant-extract dataset are not restated in the paper and are deferred to a prior reference, so exact instrument settings must be sourced separately.

## Evidence

- [results] Within the ENPKG workflow (Figure 2B, step 1), the data for each sample are processed individually to perform feature detection and export the resulting files using MZmine: [evidence span withheld — anchor status: not_found]
- [methods] Centroid mass detector noise levels for MS1 and MS2 in the ENPKG MZmine configuration.: "the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1"
- [methods] ADAP chromatogram builder parameters for ENPKG MZmine processing.: "The ADAP chromatogram builder was used and set to a minimum group size of scans of 5, minimum group intensity threshold of 1.0E4, minimum highest intensity of 5.0E5"
- [methods] Feature filtering rule that restricts retained features to those with MS2 scans and RT in 0.5-8.0 min.: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [intro] Each sample is considered individually, avoiding cross-batch RT-based feature alignment.: "Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN."
- [results] Reported feature counts across the full 1,600-plant-extract data set in both ionization modes.: "788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively"
- [readme] README grounding for the workflow repository location and its per-sample processing model.: [evidence span withheld — anchor status: external_doc]
