---
name: dims-transient-fourier-processing
description: Use when the input is direct infusion mass spectrometry (DIMS) FT-ICR
  transient .dat files organized in selected ion monitoring (SIM) windows, and a peak
  list per sample is needed before alignment, filtering and PCA.
when_to_use_negative:
- The input is already processed peak lists or an aligned feature matrix — the SimStitch
  transient pipeline is only for raw FT-ICR transient .dat files.
- The data are LC-MS profiles rather than DIMS; Galaxy-M routes LC-MS through the
  XCMS-Basic-Birmingham-Pipe (feature detection, retention time correction and alignment)
  instead of SimStitch.
- The MetaboLights accession is the only data source and lacks .dat transients — as
  with MTBLS79, the transient files must be obtained from the original authors or
  GigaDB first.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Galaxy-M
  role: Galaxy framework hosting the SimStitch tools (Sum Transients, Process Transients,
    Mass Calibration and SIM-stitching) as wrapped workflow steps
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: SimStitch
  role: Executed (under WINE, 32-bit) to sum transients, Fourier-process them, pick
    peaks at the SNR threshold, calibrate each SIM window and stitch windows into
    one peak list
- name: Galaxy
  role: Workflow platform in which the DIMS transient-processing steps are chained
    into the published 'DIMS Test Data processing' history
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:dc183c901df3c281cbec19ae22474d5d17262a71134a3d42f3de122b40e93cf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  - SimStitch
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dims-transient-fourier-processing

## Summary

Galaxy-M's DIMS raw-data processing skill: converts FT-ICR transient (.dat) files into a single calibrated, stitched peak list per sample using the SimStitch tool suite (Sum Transients, Process Transients, Mass Calibration and SIM-stitching).

## When to use

Use when the input is direct infusion mass spectrometry (DIMS) FT-ICR transient .dat files organized in selected ion monitoring (SIM) windows, and a peak list per sample is needed before alignment, filtering and PCA.

## When NOT to use

- The input is already processed peak lists or an aligned feature matrix — the SimStitch transient pipeline is only for raw FT-ICR transient .dat files.
- The data are LC-MS profiles rather than DIMS; Galaxy-M routes LC-MS through the XCMS-Basic-Birmingham-Pipe (feature detection, retention time correction and alignment) instead of SimStitch.
- The MetaboLights accession is the only data source and lacks .dat transients — as with MTBLS79, the transient files must be obtained from the original authors or GigaDB first.

## Inputs

- FT-ICR DIMS transient files in Thermo .dat format, one or more transients per SIM window per sample
- CSV configuration/instruction files from the GalaxyM-TestData folder (as referenced by the published workflows)
- SIM window definition/calibration settings expected by the SimStitch Galaxy tools

## Outputs

- Averaged transients per SIM window per sample
- Baseline-corrected, Fourier-transformed spectra per SIM window
- A single stitched and calibrated peak list per sample (m/z and intensity peaks picked at the chosen SNR threshold, e.g. 3.5:1)

## How to apply

First run Sum Transients to average each set of multiple transient data within each mass spectral SIM window. Then run Process Transients, which applies Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data, converting it into the frequency/mass domain. Next run Mass Calibration and SIM-stitching, which picks peaks with a signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert frequency-domain measurements to m/z, and stitches all SIM windows together into a single peak list per sample. In Galaxy-M these SimStitch tools run under WINE (forced 32-bit WINEARCH) and rely on the Thermo Scientific MSFileReader package to open the .dat transients; the resulting peak lists then feed Align Samples, replicate/blank/sample filtering, and PCA. Rationale: the SIM-stitching approach provides increased metabolome coverage and very high mass accuracy at ~2 min 15 s analysis time per sample, so the FT step and per-window calibration must preserve m/z accuracy for downstream MI-Pack annotation.

## Related tools

- **Galaxy-M** (Galaxy framework hosting the SimStitch tools (Sum Transients, Process Transients, Mass Calibration and SIM-stitching) as wrapped workflow steps) — https://github.com/Viant-Metabolomics/Galaxy-M
- **SimStitch** (Executed (under WINE, 32-bit) to sum transients, Fourier-process them, pick peaks at the SNR threshold, calibrate each SIM window and stitch windows into one peak list)
- **Galaxy** (Workflow platform in which the DIMS transient-processing steps are chained into the published 'DIMS Test Data processing' history) — https://github.com/galaxyproject/galaxy

## Examples

```
# In the Galaxy-M VM: load the 'DIMS Test Data processing' workflow, point it at ~/GalaxyM-TestData/DIMS_DATA .dat transients, and run Sum Transients -> Process Transients -> Mass Calibration and SIM-stitching (SNR 3.5:1)
```

## Evaluation signals

- Workflow completes in Galaxy producing a per-sample stitched peak list from the DIMS .dat transients in ~/GalaxyM-TestData/DIMS_DATA.
- Peak picking used the documented signal-to-noise threshold (e.g. 3.5:1) before calibration and stitching.
- Each SIM window was calibrated to convert frequency-domain values to m/z, and all SIM windows were stitched into one list per sample.
- Downstream alignment and filtering steps accept the peak lists without format errors, and the resulting PCA Scores Test p-value file plus MI-Pack annotation file match the published DIMS history outputs.
- No WINE 'Invalid file identifier' errors from SimStitch, which would indicate MSFileReader failed to open the DIMS data (i.e. WINE not forced to 32-bit).

## Limitations

- The transient (.dat) files are not included in the MetaboLights accession (MTBLS79) and must be obtained from the original authors or GigaDB.
- SimStitch tools run via WINE and are brittle on 64-bit Ubuntu: WINEARCH must be set to win32 or MSFileReader fails with 'Invalid file identifier' errors.
- Requires the Thermo Scientific MSFileReader package, Python 2.7 (32-bit under WINE), MATLAB Compiler Runtime 8.3, and WINE 1.7 in the tested configuration.
- The paper's DIMS processing is FT-ICR-specific (Hanning apodization, zero-filling, Fourier transformation); it does not cover other DIMS instrument types.

## Evidence

- [abstract] Sum Transients averages transient data; Process Transients does the Fourier processing: "performs Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data"
- [abstract] SIM-stitch peak picking and calibration details: "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together"
- [abstract] SimStitch performance rationale: "providing increased metabolome coverage, very high mass accuracy, and at 2 min 15 s analysis time per sample is conducive for high-throughput metabolomics"
- [readme] WINE 32-bit requirement for MSFileReader: "you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data"
- [abstract] Missing .dat transients in MetaboLights: "These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession"
