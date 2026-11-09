---
name: sim-window-mass-calibration-and-stitching
description: Use when processing FT-ICR direct infusion MS transients stored as Thermo
  .RAW/.dat files that have already been summed and apodized per SIM window, and the
  next step is calibrated m/z peak lists across all windows.
when_to_use_negative:
- The input is LC-MS data (use the XCMS-Basic-Birmingham-Pipe feature detection /
  alignment path instead, not SIM-stitching).
- The FT-ICR transients have not yet been summed and processed (apodization / Fourier
  transform / baseline correction) — run Sum Transients and Process Transients first.
- You only need raw frequency-domain spectra or have not set up WINE with WINEARCH=win32
  and MSFileReader; the tools cannot open Thermo data without that environment.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M Sum Transients
  role: 'Upstream tool: averages each set of multiple transient data within each mass
    spectral SIM window so calibration can be done on averaged spectra.'
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy-M Process Transients
  role: 'Upstream tool: applies Hanning apodization, zero-filling, Fourier transformation
    and baseline correction to the averaged FT-ICR transients, producing the frequency-domain
    spectra this tool calibrates.'
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Mass Calibration and SIM-stitching
  role: 'The core tool: picks peaks with a signal-to-noise-ratio threshold (e.g. 3.5:1),
    converts each SIM window''s frequency domain to m/z, and stitches all windows
    into a single peak list.'
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Workflow platform in which the Sum Transients / Process Transients / SIM-stitching
    steps are executed.
  repo: https://github.com/galaxyproject/galaxy/
- name: Thermo Scientific MSFileReader package
  role: Windows dependency used inside WINE by the SimStitch tools to open Thermo
    .RAW/.dat data files.
  repo: https://thermo.flexnetoperations.com
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:27d073a2429be48b044145f708e30803267a0990dc23f3b7f5e64db921fe138a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M Process Transients
  - Galaxy-M Sum Transients
  - Mass Calibration and SIM-stitching
  - Thermo Scientific MSFileReader package
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sim-window-mass-calibration-and-stitching

## Summary

In the Galaxy-M DIMS pipeline, Mass Calibration and SIM-stitching converts each FT-ICR SIM window's frequency-domain spectra to m/z measurements, peak picks with an SNR threshold, and stitches all SIM windows into a single peak list.

## When to use

Use when processing FT-ICR direct infusion MS transients stored as Thermo .RAW/.dat files that have already been summed and apodized per SIM window, and the next step is calibrated m/z peak lists across all windows.

## When NOT to use

- The input is LC-MS data (use the XCMS-Basic-Birmingham-Pipe feature detection / alignment path instead, not SIM-stitching).
- The FT-ICR transients have not yet been summed and processed (apodization / Fourier transform / baseline correction) — run Sum Transients and Process Transients first.
- You only need raw frequency-domain spectra or have not set up WINE with WINEARCH=win32 and MSFileReader; the tools cannot open Thermo data without that environment.

## Inputs

- Baseline-corrected FT-ICR frequency-domain spectra, one spectrum per SIM window per sample (output of the Galaxy-M Process Transients tool)
- Thermo instrument data files (.RAW / .dat transients) read via MSFileReader in WINE
- A signal-to-noise-ratio threshold parameter (e.g. 3.5:1)

## Outputs

- A stitched peak list of m/z values per sample spanning all SIM windows
- Per-window calibrated m/z values derived from the frequency domain

## How to apply

After the Galaxy-M Sum Transients tool averages each set of multiple transients within each mass spectral SIM window and the Process Transients tool applies Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data, run Mass Calibration and SIM-stitching on the resulting spectra. The tool picks peaks using a signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements, and stitches all the SIM windows together into one peak list of m/z values. Because the tool reads Thermo data files via MSFileReader, the execution environment must have WINE configured in 32-bit mode (WINEARCH=win32) and the Thermo MSFileReader package installed in WINE; otherwise SimStitch errors such as 'Invalid file identifier' occur. After stitching, downstream quality-control steps like Replicate Filtering and Align Samples operate on the stitched m/z peak list. High mass accuracy and the stitched peak list are what make the subsequent align-across-samples step possible.

## Related tools

- **Galaxy-M Sum Transients** (Upstream tool: averages each set of multiple transient data within each mass spectral SIM window so calibration can be done on averaged spectra.) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy-M Process Transients** (Upstream tool: applies Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transients, producing the frequency-domain spectra this tool calibrates.) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Mass Calibration and SIM-stitching** (The core tool: picks peaks with a signal-to-noise-ratio threshold (e.g. 3.5:1), converts each SIM window's frequency domain to m/z, and stitches all windows into a single peak list.) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Workflow platform in which the Sum Transients / Process Transients / SIM-stitching steps are executed.) — https://github.com/galaxyproject/galaxy/
- **Thermo Scientific MSFileReader package** (Windows dependency used inside WINE by the SimStitch tools to open Thermo .RAW/.dat data files.) — https://thermo.flexnetoperations.com

## Examples

```
export WINEARCH=win32; wine ~/.wine/drive_c/Program\ Files\ (x86)/SimStitch/SimStitch.exe --input processed_transients.dat --snr 3.5
```

## Evaluation signals

- The output is a single stitched peak list of m/z values spanning all SIM windows, not per-window fragments.
- Peaks below the configured SNR threshold (e.g. 3.5:1) are absent from the stitched peak list.
- Downstream Replicate Filtering and Align Samples run successfully on the stitched m/z peak list without missing-window artifacts.
- No 'Invalid file identifier' errors — confirming WINEARCH=win32 and MSFileReader were configured so Thermo data opened correctly.
- Frequency-domain-to-m/z calibration yields high mass accuracy (the article cites very high mass accuracy as a hallmark of the SIM-stitching DIMS workflow).

## Limitations

- Requires WINE in 32-bit mode (WINEARCH=win32) and Thermo MSFileReader; misconfiguration produces 'Invalid file identifier' errors when the tools cannot use MSFileReader to open the DIMS data.
- The DIMS transient (.dat) test data are not included in the MetaboLights accession (MTBLS79) and had to be obtained from the original authors.
- The SNR threshold is user-chosen (e.g. 3.5:1); an inappropriate threshold will change which peaks survive into the stitched list.

## Evidence

- [abstract] calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together: [evidence span withheld — anchor status: not_found]
- [abstract] picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1): "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together"
- [readme] You must force wine to install everything in 32bit mode.: "IMPORTANT: You must force wine to install everything in 32bit mode. In order to get WINE to run in 32bit, remove the .wine folder in your home directory... Then set the ``WINEARCH`` system variable"
- [readme] you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier': "if you do not do this, you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data."
- [abstract] These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession: "These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession"
