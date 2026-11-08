---
name: mgf-spectra-export
description: Use when starting from Thermo RAW (or mzML/mzXML) LC-HRMS2 DDA files
  that must be converted into a feature quantification table plus per-feature MS2
  .mgf spectra for each sample individually, before annotation or molecular networking.
when_to_use_negative:
- Input is already a feature quantification table or .mgf spectra file (feature detection
  and export have already been done).
- The analysis goal requires a classical data set-centric RT/m/z feature alignment
  across all samples rather than per-sample processing — ENPKG deliberately processes
  each sample individually.
- Data were not acquired as LC-HRMS2 DDA (e.g. DIA or direct infusion), since the
  noise thresholds (1.0E4 MS1, 0 MS2) and MS2-scan filter assume DDA spectra.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MZmine
  role: Performs mass detection (centroid, noise 1.0E4 MS1 / 0 MS2), ADAP chromatogram
    building, wavelets (ADAP) deconvolution, isotope grouping, feature filtering,
    and export of the quantification table and .mgf MS2 spectra via the 'Export to
    GNPS' module
  repo: http://mzmine.github.io/
- name: ProteoWizard
  role: Converts Thermo RAW LC-MS files to mzXML/mzML as the entry point for MZmine
    processing
- name: matchms
  role: Downstream consumer of the exported .mgf spectra for matchms-based ISDB spectral
    annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:f76c379d96ee8acee18bf48e77bcf3394225736f966d6b15807e15333c645a69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mgf-spectra-export

## Summary

Per-sample MZmine feature detection and export of deconvoluted LC-HRMS2 features plus their MS2 spectra as .mgf files (via the 'Export to GNPS' module), so downstream spectral matching (matchms-based ISDB) and molecular networking steps can consume them.

## When to use

Use when starting from Thermo RAW (or mzML/mzXML) LC-HRMS2 DDA files that must be converted into a feature quantification table plus per-feature MS2 .mgf spectra for each sample individually, before annotation or molecular networking.

## When NOT to use

- Input is already a feature quantification table or .mgf spectra file (feature detection and export have already been done).
- The analysis goal requires a classical data set-centric RT/m/z feature alignment across all samples rather than per-sample processing — ENPKG deliberately processes each sample individually.
- Data were not acquired as LC-HRMS2 DDA (e.g. DIA or direct infusion), since the noise thresholds (1.0E4 MS1, 0 MS2) and MS2-scan filter assume DDA spectra.

## Inputs

- Thermo RAW LC-MS/MS DDA files converted to mzXML/mzML (e.g. via MSConvert)
- Per-sample minimal metadata file (originating taxon) as required by the enpkg_workflow
- MZmine feature lists (positive and/or negative ionization modes)

## Outputs

- Deconvoluted feature quantification table (m/z, RT, area/intensity per feature)
- Per-feature MS2 spectra exported as .mgf files (raw LC-HRMS2 .mzML plus .mgf, ready for 'Export to GNPS' consumption by FBMN/matchms-ISDB)

## How to apply

Convert Thermo RAW files to mzXML/mzML with MSConvert (ProteoWizard), then in MZmine detect masses with the centroid mass detector (noise level 1.0E4 for MS level 1, 0 for MS level 2), build chromatograms with the ADAP chromatogram builder (minimum group size 5 scans, minimum group intensity 1.0E4, minimum highest intensity 5.0E5, 12 ppm m/z tolerance), and deconvolute with wavelets (ADAP) using the intensity window S/N estimator (S/N 10, minimum feature height 5.0E5, coefficient area threshold 50, peak duration 0.02–0.5 min, RT wavelet range 0.01–0.03 min). Group isotopes with the isotope peaks grouper (12 ppm m/z tolerance, 0.01 min absolute RT tolerance, max charge 2, most intense isotope as representative). Filter each feature list to keep only features with an associated MS2 scan and RT between 0.5 and 8.0 min. Finally export the deconvoluted feature quantification table and per-feature MS2 spectra as .mgf files with the 'Export to GNPS' module; in ENPKG this is done per sample to avoid RT-based feature alignment across batches.

## Related tools

- **MZmine** (Performs mass detection (centroid, noise 1.0E4 MS1 / 0 MS2), ADAP chromatogram building, wavelets (ADAP) deconvolution, isotope grouping, feature filtering, and export of the quantification table and .mgf MS2 spectra via the 'Export to GNPS' module) — http://mzmine.github.io/
- **ProteoWizard** (Converts Thermo RAW LC-MS files to mzXML/mzML as the entry point for MZmine processing)
- **matchms** (Downstream consumer of the exported .mgf spectra for matchms-based ISDB spectral annotation)

## Examples

```
sh workflow/00_workflow_all.sh
```

## Evaluation signals

- The exported feature quantification table exists with the expected columns (m/z, RT, area/intensity per feature).
- The exported .mgf files contain an MS2 spectrum for each retained feature (i.e. every feature in the table has an associated MS2 scan).
- No retained feature has a retention time outside 0.5–8.0 min (feature filtering module criterion).
- Feature counts are consistent with the reported scale (e.g. ~788,623 features in PI and ~364,967 in NI for the full 1,600-extract data set before per-sample filtering).
- Each sample is processed and exported individually (sample-wise folder architecture), not through a data set-level RT alignment.

## Limitations

- Classical RT-based feature-alignment steps across samples hinder comparison of samples from different batches or instrumental setups — the reason ENPKG exports per sample instead.
- Minimum feature height of 5.0E5 and S/N 10 thresholds were tuned for these Q-ToF/Orbitrap plant-extract data; other instruments or matrices may need recalibration.
- Features without an associated MS2 scan are discarded, so MS1-only features will not appear in the .mgf export.

## Evidence

- [results] Within the ENPKG workflow (Figure 2B, step 1), the data for each sample are processed individually to perform feature detection and export the resulting files using MZmine: "the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine."
- [methods] Feature filtering keeps only MS2-backed features in the 0.5-8.0 min RT window before export.: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [methods] Mass detection parameters: centroid detector with noise level 1.0E4 for MS level 1.: "the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1"
- [readme] The enpkg_workflow processes each sample individually and relies on MZmine before downstream annotation steps.: [evidence span withheld — anchor status: external_doc]
