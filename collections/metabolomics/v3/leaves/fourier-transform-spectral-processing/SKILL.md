---
name: fourier-transform-spectral-processing
description: Use when raw FT-ICR direct-infusion mass spectrometry (DIMS) data are
  time-domain transients per SIM window and you need calibrated, baseline-corrected
  spectra before peak picking and m/z assignment.
when_to_use_negative:
- Input is already LC-MS chromatographic data — use the XCMS-Basic-Birmingham-Pipe
  (feature detection, retention time correction, alignment) instead of the transient-processing
  chain
- Data are already frequency-domain or m/z peak lists — skip directly to Mass Calibration
  and SIM-stitching or Align Samples
- Only processed peak tables are available and the original transient files cannot
  be obtained
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M Process Transients
  role: Applies Hanning apodization, zero-filling, Fourier transformation and baseline
    correction to the averaged FT-ICR transient data
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy-M Sum Transients
  role: Upstream step that averages each set of multiple transients within each mass
    spectral SIM window, producing the input to this skill
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy-M (Mass Calibration and SIM-stitching)
  role: Downstream consumer that picks peaks (S/N ~3.5:1), converts frequency domain
    to m/z and stitches the SIM windows
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:d018dbf5ec6621c6e531f9fcdc914d1547c4f55061d60f60cc43ca57a002bfc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M (Mass Calibration and SIM-stitching)
  - Galaxy-M Process Transients
  - Galaxy-M Sum Transients
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fourier-transform-spectral-processing

## Summary

A Galaxy-M DIMS processing step that converts averaged FT-ICR time-domain transients into frequency-domain spectra via Hanning apodization, zero-filling, Fourier transformation and baseline correction, preparing them for m/z calibration and SIM-window stitching.

## When to use

Use when raw FT-ICR direct-infusion mass spectrometry (DIMS) data are time-domain transients per SIM window and you need calibrated, baseline-corrected spectra before peak picking and m/z assignment.

## When NOT to use

- Input is already LC-MS chromatographic data — use the XCMS-Basic-Birmingham-Pipe (feature detection, retention time correction, alignment) instead of the transient-processing chain
- Data are already frequency-domain or m/z peak lists — skip directly to Mass Calibration and SIM-stitching or Align Samples
- Only processed peak tables are available and the original transient files cannot be obtained

## Inputs

- Averaged FT-ICR transient data per SIM window (output of the Galaxy-M Sum Transients tool)
- Thermo .RAW/.dat DIMS transient files (e.g. from MetaboLights MTBLS79 test data)

## Outputs

- Frequency-domain spectra per SIM window (apodized, zero-filled, Fourier-transformed, baseline-corrected)
- Downstream peak list of m/z values after Mass Calibration and SIM-stitching

## How to apply

First run the Galaxy-M Sum Transients tool to average the multiple transients recorded within each mass spectral SIM window for every sample. Then run the Process Transients tool on those averaged transients: it applies Hanning apodization, zero-filling, Fourier transformation and baseline correction to produce frequency-domain spectra. The tool is part of the Viant-Metabolomics Galaxy-M repository and executes via the MATLAB Compiler Runtime (MCR 8.3) or MATLAB 2014a, so ensure the runtime is installed. The resulting frequency-domain spectra are consumed downstream by the Mass Calibration and SIM-stitching tool, which picks peaks at a signal-to-noise threshold of about 3.5:1 and converts the frequency domain to m/z. Verify correctness by checking that stitched peak lists emerge from all SIM windows and survive Replicate Filtering (x-out-of-n technical replicates) and Align Samples.

## Related tools

- **Galaxy-M Process Transients** (Applies Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy-M Sum Transients** (Upstream step that averages each set of multiple transients within each mass spectral SIM window, producing the input to this skill) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy-M (Mass Calibration and SIM-stitching)** (Downstream consumer that picks peaks (S/N ~3.5:1), converts frequency domain to m/z and stitches the SIM windows) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- The averaged-transient input must cover every SIM window for every sample before processing
- Each output spectrum should show Hanning-apodized, zero-filled, baseline-corrected frequency content that the Mass Calibration and SIM-stitching tool can convert to m/z
- Downstream peak picking with the ~3.5:1 signal-to-noise threshold should yield peaks across all stitched SIM windows
- Resulting peaks should pass Replicate Filtering (appearing in at least x-out-of-n technical replicates) and Align Samples should align them across all samples into a CSV peak table
- The DIMS workflow should retain the reported high mass accuracy and increased metabolome coverage of the SIM-stitching approach

## Limitations

- The DIMS transient (.dat) files are not included in the MetaboLights accession (MTBLS79) and must be obtained from the original authors
- The processing chain requires Wine installed in 32-bit mode (WINEARCH=win32) plus Thermo MSFileReader 3.0 sp2 so the SimStitch tools can open the Thermo .RAW/.dat data files; otherwise errors like 'Invalid file identifier' occur
- The tool is MATLAB-based and needs the MATLAB Compiler Runtime (MCR 8.3) or MATLAB 2014a to execute

## Evidence

- [abstract] Process Transients: Hanning apodization, zero-filling, Fourier transformation and baseline correction: "performs Hanning apodization, zero-filling, Fourier transformation and baseline correction to the averaged FT-ICR transient data"
- [abstract] Sum Transients: averaging transients in each SIM window: "Sum Transients: average each set of multiple transient data within each mass spectral SIM window"
- [readme] MCR requirement for the MATLAB-compiled tools: "The MATLAB Runtime is a standalone set of shared libraries that enables the execution of compiled MATLAB applications or components on computers that do not have MATLAB installed."
- [abstract] Transient .dat files not in MetaboLights accession: "These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession"
