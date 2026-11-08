---
name: ft-icr-transient-averaging
description: Use when direct infusion FT-ICR-MS data arrives as multiple transient
  acquisitions per SIM window per sample and must be averaged before frequency-domain
  processing.
when_to_use_negative:
- Input is already frequency-domain spectra or an m/z peak list — averaging applies
  only to raw transient acquisitions
- Data are LC-MS profiles; the Galaxy-M LC-MS branch uses XCMS feature detection rather
  than transient averaging
- Each SIM window contains only a single transient acquisition, so there is nothing
  to average
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Galaxy-M Sum Transients
  role: Averages each set of multiple transient data within each mass spectral SIM
    window for every sample
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Thermo Scientific MSFileReader
  role: Windows library accessed through WINE that lets the SimStitch tool chain open
    the Thermo .RAW/.dat transient files
- name: Wine (1.7)
  role: Compatibility layer used to run the Thermo-dependent tools; must be forced
    to 32-bit mode (WINEARCH=win32)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:70526a4ed7ec0f687250b04004bb4759eb5caa3f8de89fc55a7009f3ca0300c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M Sum Transients
  - Thermo Scientific MSFileReader
  - Wine (1.7)
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# FT-ICR transient averaging

## Summary

Averages multiple FT-ICR transient acquisitions within each mass spectral SIM window per sample (Galaxy-M 'Sum Transients'), producing a single averaged transient per SIM window that is passed forward for apodization, Fourier transformation and SIM-stitching.

## When to use

Use when direct infusion FT-ICR-MS data arrives as multiple transient acquisitions per SIM window per sample and must be averaged before frequency-domain processing.

## When NOT to use

- Input is already frequency-domain spectra or an m/z peak list — averaging applies only to raw transient acquisitions
- Data are LC-MS profiles; the Galaxy-M LC-MS branch uses XCMS feature detection rather than transient averaging
- Each SIM window contains only a single transient acquisition, so there is nothing to average

## Inputs

- Multiple FT-ICR transient acquisitions per SIM window per sample (Thermo .RAW/.dat DIMS files)
- SIM window definition/instruction CSV files used by the Galaxy-M DIMS workflow

## Outputs

- One averaged FT-ICR transient per SIM window per sample, ready for Process Transients (Hanning apodization, zero-filling, FFT, baseline correction)

## How to apply

Run the Galaxy-M Sum Transients tool to average each set of multiple transient data within each mass spectral SIM window for every sample, reading the Thermo .RAW/.dat DIMS transients (e.g. the MTBLS79 subset used in Galaxy-M-TestData/DIMS_DATA). Because the tool chain opens Thermo files via MSFileReader, the execution environment must be prepared first: install WINE in forced 32-bit mode (WINEARCH=win32) and the Thermo MSFileReader 3.0 sp2 package, otherwise the SimStitch tools fail with 'Invalid file identifier'. The output averaged transients then feed the Process Transients tool (Hanning apodization, zero-filling, Fourier transformation, baseline correction) before Mass Calibration and SIM-stitching converts frequency domain to m/z. Correctness is judged by confirming each SIM window of each sample yields exactly one averaged transient and that downstream peak picking (S/N threshold ~3.5:1) succeeds without file-access errors.

## Related tools

- **Galaxy-M Sum Transients** (Averages each set of multiple transient data within each mass spectral SIM window for every sample) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Thermo Scientific MSFileReader** (Windows library accessed through WINE that lets the SimStitch tool chain open the Thermo .RAW/.dat transient files)
- **Wine (1.7)** (Compatibility layer used to run the Thermo-dependent tools; must be forced to 32-bit mode (WINEARCH=win32))

## Examples

```
export WINEARCH=win32  # required 32-bit WINE so the Galaxy-M Sum Transients/SimStitch tools can open Thermo .RAW/.dat DIMS transients
```

## Evaluation signals

- Each SIM window of each sample yields exactly one averaged transient (no windows dropped or duplicated)
- Downstream Process Transients runs without 'Invalid file identifier' errors, confirming WINE/MSFileReader were correctly configured
- Mass Calibration and SIM-stitching subsequently picks peaks at the ~3.5:1 signal-to-noise threshold, indicating the averaged transients were valid
- Averaged transients match the expected FT-ICR time-domain structure for the DIMS test data (GalaxyM-TestData/DIMS_DATA)

## Limitations

- The DIMS transient (.dat) files were not included in the MetaboLights accession and had to be obtained from the original authors
- The tool chain depends on WINE in 32-bit mode plus the proprietary Thermo MSFileReader package; installing 64-bit WINE causes 'Invalid file identifier' errors when opening DIMS data
- Averaging is only the first DIMS stage — its output is unusable without the subsequent Process Transients and Mass Calibration/SIM-stitching steps

## Evidence

- [abstract] Sum Transients averages transients per SIM window: "Sum Transients: average each set of multiple transient data within each mass spectral SIM window"
- [abstract] Process Transients operates on the averaged FT-ICR transients: "performs Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data"
- [readme] WINE must be forced into 32-bit mode for the SimStitch tools to open DIMS data: "You must force wine to install everything in 32bit mode."
- [readme] SimStitch tools fail without 32-bit WINE/MSFileReader: "you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data"
- [readme] Test DIMS transients location on the pre-installed VM: "the Mass Spectra can be found in :

* `~/GalaxyM-TestData/LCMS_DATA`
* `~/GalaxyM-TestData/DIMS_DATA`"
