---
name: msi-peak-matrix-processing
description: Use when raw MSI spectral data (imzML in profile mode) must be converted
  into a centroid-mode peak matrix and processed dataset suitable for matrix-related
  peak annotation.
when_to_use_negative:
- Input is already a rMSIproc-format peak matrix (.zip) and processed file (.tar)
  — e.g. the rMSIcleanup Sample Data Ag_Pancreas_TOF_2015_Dataset2 — which can be
  loaded directly.
- The goal is only to annotate or remove matrix-related peaks on an existing processed
  peak matrix; use rMSIcleanup's annotate_matrix / remove_matrix instead of re-processing
  spectra.
- Data normalization is required before downstream analysis — the paper's pipeline
  explicitly performs no normalization.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIproc
  role: Executes the complete MSI pre-processing pipeline (smoothing, alignment, calibration,
    peak-picking, peak-binning) and generates the peak matrix; invoked via rMSIproc::ProcessWizard()
    and output loaded with rMSIproc::LoadPeakMatrix
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Provides the efficient data-access layer and the rMSI (.tar) storage format
    that rMSIproc builds on; also used to load the processed dataset with rMSI::LoadMsiData
- name: R
  role: Runtime environment in which rMSI and rMSIproc are installed via devtools::install_github
    and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:d2047c4ecc124b0841ac522958d190a75cc1572aefbadd14a5948584cb256026
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prafols/rMSIproc
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# msi-peak-matrix-processing

## Summary

Process raw MSI spectral data into a centroid-mode peak matrix using the rMSIproc pipeline, so that downstream annotation tools such as rMSIcleanup can consume a peak matrix (.zip) alongside processed spectra (.tar).

## When to use

Use when raw MSI spectral data (imzML in profile mode) must be converted into a centroid-mode peak matrix and processed dataset suitable for matrix-related peak annotation.

## When NOT to use

- Input is already a rMSIproc-format peak matrix (.zip) and processed file (.tar) — e.g. the rMSIcleanup Sample Data Ag_Pancreas_TOF_2015_Dataset2 — which can be loaded directly.
- The goal is only to annotate or remove matrix-related peaks on an existing processed peak matrix; use rMSIcleanup's annotate_matrix / remove_matrix instead of re-processing spectra.
- Data normalization is required before downstream analysis — the paper's pipeline explicitly performs no normalization.

## Inputs

- Raw MSI spectral data exported to imzML in profile mode
- XMASS or rMSI-format (TAR) MSI datasets (alternative supported input formats)

## Outputs

- Processed MSI dataset stored in rMSI format (.tar), loadable with rMSI::LoadMsiData
- Peak matrix in centroid mode stored as .zip, loadable with rMSIproc::LoadPeakMatrix

## How to apply

Run rMSIproc::ProcessWizard() (or the equivalent R calls) on the exported imzML profile-mode file to execute the full pre-processing pipeline: Savitzky–Golay smoothing with kernel size 7, SNR threshold 5, mass re-calibration using Ag reference peaks, peak-picking, and peak binning where peaks within a window of 6 data-points or scans are binned together. Filter out peaks appearing in less than 5% of the pixels and store the result as a processed MSI dataset in rMSI format (.tar) plus a peak matrix (.zip) readable by rMSIproc. The rationale is that rMSIcleanup's annotate_matrix and remove_matrix functions require both the peak matrix (loaded via rMSIproc::LoadPeakMatrix) and the processed spectra (loaded via rMSI::LoadMsiData), so the peak matrix must be generated first. No data normalization should be applied at this stage, per the paper's methods.

## Related tools

- **rMSIproc** (Executes the complete MSI pre-processing pipeline (smoothing, alignment, calibration, peak-picking, peak-binning) and generates the peak matrix; invoked via rMSIproc::ProcessWizard() and output loaded with rMSIproc::LoadPeakMatrix) — https://github.com/prafols/rMSIproc
- **rMSI** (Provides the efficient data-access layer and the rMSI (.tar) storage format that rMSIproc builds on; also used to load the processed dataset with rMSI::LoadMsiData)
- **R** (Runtime environment in which rMSI and rMSIproc are installed via devtools::install_github and executed)

## Examples

```
rMSIproc::ProcessWizard(); pks <- rMSIproc::LoadPeakMatrix("peakmatrix.zip"); full <- rMSI::LoadMsiData("processed.tar")
```

## Evaluation signals

- The output peak matrix (.zip) and processed dataset (.tar) exist and are loadable with rMSIproc::LoadPeakMatrix and rMSI::LoadMsiData respectively.
- Peaks appearing in less than 5% of the pixels are absent from the resulting peak matrix.
- No peaks within a window of 6 data-points or scans remain as separate entries — they are binned as a single mass peak.
- Savitzky–Golay smoothing was applied with kernel size 7 and the SNR threshold was set to 5, matching the paper's processing parameters.
- Mass spectra were re-calibrated using the Ag reference peaks as reference masses.

## Limitations

- rMSIproc's internal processing is implemented in C++ and requires the fftw library on Linux (on Windows the fft library ships as redistributable binaries inside the package).
- rMSIcleanup consumes rMSIproc-format data only — to annotate your own data you must first process the imzML through rMSIproc.
- The documented pipeline performs no data normalization, which may affect downstream comparative analyses if normalization is expected.

## Evidence

- [methods] Peak matrix generation in centroid mode with rMSIproc: "The software rMSIproc [20] was used to process the data and generate a peak matrix in centroid mode."
- [methods] Filter peaks appearing in less than 5% of pixels: "Peaks appearing in less than 5% of the pixels were filtered out."
- [methods] Peak binning window of 6 data-points or scans: "Peaks within a window of 6 data-points or scans were binned together as the same mass peak."
- [methods] Savitzky–Golay smoothing and SNR threshold: "The Signal-to-Noise Ratio (SNR) threshold was set to 5 and the Savitzky–Golay smoothing had a kernel size of 7."
- [readme] Load sample peak matrix and processed spectra: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
- [readme] Process imzML via ProcessWizard: "rMSIcleanup uses data in the rMSIproc format. To annotate your own data you will have to process the imzML"
- [methods] Export raw spectral data to imzML in profile mode: "The raw spectral data of each MSI dataset was exported to the imzML data format [19] in profile mode."
- [methods] No data normalization performed: "No data normalization was performed."
