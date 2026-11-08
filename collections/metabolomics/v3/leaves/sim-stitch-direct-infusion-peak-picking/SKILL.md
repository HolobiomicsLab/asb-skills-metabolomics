---
name: sim-stitch-direct-infusion-peak-picking
description: Use when processing direct infusion mass spectrometry (DIMS) FT-ICR data
  stored as .dat transient files organized in SIM windows (e.g. MTBLS79 DIMS_DATA)
  and you need a calibrated, stitched peak list across samples.
when_to_use_negative:
- The input is LC-MS data — use the XCMS-Basic-Birmingham-Pipe route (feature detection,
  retention time correction and alignment) instead of SIM-stitching
- The data are already peak-picked and aligned (a feature/peak table) — start at Replicate
  Filtering, Blank Filtering and Sample Filtering
- The .dat transient files are unavailable — they are not included in the MetaboLights
  MTBLS79 accession and must be obtained from the original authors
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Galaxy-M
  role: Platform hosting the SimStitch tools (Sum Transients, Process Transients,
    Mass Calibration and SIM-stitching, Align Samples) as wrapped Galaxy tools
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: SimStitch
  role: Set of tools performing transient averaging, apodization/FT/baseline correction,
    m/z calibration, SNR-threshold peak picking and SIM-window stitching
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:a9c8ebb5aa47054ec6b137c14ac80f7606fdbb20c9d3630f48b54e4b59d8d247
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M
  - PLS-Toolbox
  - SimStitch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SIM-stitch direct infusion peak picking

## Summary

DIMS peak-picking pipeline (SimStitch tools in Galaxy-M) that converts averaged FT-ICR transients per SIM window into a single calibrated, stitched peak list per sample, providing increased metabolome coverage and very high mass accuracy at ~2 min 15 s analysis time per sample.

## When to use

Use when processing direct infusion mass spectrometry (DIMS) FT-ICR data stored as .dat transient files organized in SIM windows (e.g. MTBLS79 DIMS_DATA) and you need a calibrated, stitched peak list across samples.

## When NOT to use

- The input is LC-MS data — use the XCMS-Basic-Birmingham-Pipe route (feature detection, retention time correction and alignment) instead of SIM-stitching
- The data are already peak-picked and aligned (a feature/peak table) — start at Replicate Filtering, Blank Filtering and Sample Filtering
- The .dat transient files are unavailable — they are not included in the MetaboLights MTBLS79 accession and must be obtained from the original authors

## Inputs

- DIMS FT-ICR .dat transient files (e.g. from ~/GalaxyM-TestData/DIMS_DATA, MTBLS79 cow/sheep heart subset with QC and blank)
- CSV instruction/config files from the GalaxyM-TestData folder as used by the published workflow
- Signal-to-noise ratio threshold (e.g. 3.5:1)

## Outputs

- One calibrated peak list per sample with all SIM windows stitched together
- A cross-sample aligned peak matrix (after the Align Samples step) feeding Replicate/Blank/Sample Filtering

## How to apply

In the Galaxy-M VM, run the SimStitch tools in sequence on the DIMS .dat transient files: (1) Sum Transients to average the multiple transients within each mass spectral SIM window; (2) Process Transients to apply Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transients; (3) Mass Calibration and SIM-stitching to pick peaks with a signal-to-noise ratio threshold (e.g. 3.5:1), calibrate each SIM window from the frequency domain to m/z, and stitch all SIM windows into one peak list. Follow with Align Samples to align the resulting peaks across all samples. The SimStitch tools are Windows executables run under WINE with Thermo MSFileReader, so WINE must be forced to 32-bit mode (WINEARCH=win32) or the tools fail to open the DIMS data with 'Invalid file identifier' errors.

## Related tools

- **Galaxy-M** (Platform hosting the SimStitch tools (Sum Transients, Process Transients, Mass Calibration and SIM-stitching, Align Samples) as wrapped Galaxy tools) — https://github.com/Viant-Metabolomics/Galaxy-M
- **SimStitch** (Set of tools performing transient averaging, apodization/FT/baseline correction, m/z calibration, SNR-threshold peak picking and SIM-window stitching) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- Each SIM window's peaks are converted from the frequency domain to m/z via calibration before stitching — verify the stitched list has m/z, not frequency values
- Peaks below the chosen SNR threshold (e.g. 3.5:1) should be absent from the picked peak list
- Peak lists from all SIM windows and all samples should be represented in the aligned output matrix
- The SimStitch tools complete without 'Invalid file identifier' errors (confirms WINE 32-bit mode and MSFileReader can open the .dat transients)
- Downstream DSO creation and PCA on the aligned matrix should succeed, consistent with the published 'DIMS Test Data processing' history in the Galaxy-M VM

## Limitations

- SIM-stitching requires .dat transient files which are not included in the MetaboLights accession and must be obtained from the original authors
- SimStitch runs Windows executables under WINE; on 64-bit Ubuntu, WINE must be forced to 32-bit (WINEARCH=win32) or MSFileReader cannot open the DIMS data
- The published DIMS workflow runs at ~2 min 15 s per sample but the tool versions are pinned (Galaxy commit c429777c93680dcee449fe410f5360afbe673758; MI-Pack commit 06ce0ace643ee1cf1d27550769a5272b7ea50825; PLS-Toolbox 7.0.3/8.0.2), so newer environments may not reproduce results exactly

## Evidence

- [abstract] Process Transients: Hanning apodization, zero-filling, Fourier transformation and baseline correction: "performs Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data"
- [abstract] Mass Calibration and SIM-stitching SNR threshold: "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together"
- [abstract] SimStitch performance claim: "providing increased metabolome coverage, very high mass accuracy, and at 2 min 15 s analysis time per sample is conducive for high-throughput metabolomics"
- [readme] WINE 32-bit requirement for SimStitch: "if you do not do this, you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data"
- [abstract] Test data location and transient availability: "These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession"
