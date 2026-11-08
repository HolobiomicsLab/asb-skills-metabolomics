---
name: spectral-peak-detection-with-snr-thresholding
description: Use when you have a baseline-corrected spectrum (m/z and intensity columns)
  and need to discard noise features by flagging peaks whose intensity divided by
  segment-level noise falls below a user-specified SNR_thresh.
when_to_use_negative:
- Input spectrum is raw and has not yet been baseline-corrected — the SNR thresholding
  is designed to operate on the output of subMALDI's baseline correction step
- Input is already an aligned feature/intensity matrix (subMALDI's alignment step
  aligns processed spectra to an array of all theoretically possible m/z values at
  a specified resolution, which happens after peak detection)
- The workflow requires S3/S4 class object infrastructure, e.g. MALDIquant- or MSnbase-style
  containers — subMALDI intentionally keeps m/z and intensity data unmasked in plain
  data frames
tools:
- name: subMALDI
  role: R package providing the SNR-based peak detection function, segment-MAD noise
    estimation, and slope-based endpoint filtering applied in this skill
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which the subMALDI peak detection workflow is executed
- name: MassSpecWavelet
  role: BioConductor dependency that must be installed before subMALDI can be installed
    from source
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:1d828d0b44b87db951c83bb2e5a483832aaabaee9c0c8ae02d30376aa418d99f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-peak-detection-with-snr-thresholding

## Summary

Detects peaks in mass spectrometry spectra by thresholding each candidate peak's signal-to-noise ratio (SNR), where noise is estimated as the median absolute deviation (MAD) of intensity within fixed-size spectral segments, as implemented in the subMALDI R package.

## When to use

Use when you have a baseline-corrected spectrum (m/z and intensity columns) and need to discard noise features by flagging peaks whose intensity divided by segment-level noise falls below a user-specified SNR_thresh.

## When NOT to use

- Input spectrum is raw and has not yet been baseline-corrected — the SNR thresholding is designed to operate on the output of subMALDI's baseline correction step
- Input is already an aligned feature/intensity matrix (subMALDI's alignment step aligns processed spectra to an array of all theoretically possible m/z values at a specified resolution, which happens after peak detection)
- The workflow requires S3/S4 class object infrastructure, e.g. MALDIquant- or MSnbase-style containers — subMALDI intentionally keeps m/z and intensity data unmasked in plain data frames

## Inputs

- Baseline-corrected spectrum from subMALDI's baseline correction step, as a data frame of m/z and intensity values
- User-specified SNR_thresh (signal-to-noise ratio threshold for retaining peaks)
- User-specified segment size n (number of spectral points per segment for MAD noise estimation)

## Outputs

- Peak-filtered spectrum: detected peaks retained after SNR thresholding and slope-based endpoint filtering, kept as a plain m/z–intensity data frame suitable for downstream alignment and normalization

## How to apply

Load the baseline-corrected spectrum produced by subMALDI's baseline correction step as a data frame of m/z and intensity values; subMALDI deliberately avoids masking data into S3 or S4 class objects so the values stay accessible at each step. Divide the spectrum into segments of user-specified size n and compute the noise in each segment as the median absolute deviation (MAD) of the intensities. Run subMALDI's peak detection function with the user-specified SNR_thresh so that peak candidates are flagged by the ratio of peak intensity to segment noise; candidates whose intensity/noise falls below SNR_thresh are discarded. Apply the package's slope-based endpoint filtering to the flagged candidates to discard features that do not have the characteristic shape of real peaks. Finally, inspect the baseline-corrected spectrum for peaks that fell below the SNR threshold and confirm each one is absent from the peak-filtered output before saving it for downstream alignment and normalization.

## Related tools

- **subMALDI** (R package providing the SNR-based peak detection function, segment-MAD noise estimation, and slope-based endpoint filtering applied in this skill) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which the subMALDI peak detection workflow is executed)
- **MassSpecWavelet** (BioConductor dependency that must be installed before subMALDI can be installed from source)

## Evaluation signals

- Acceptance check from the package's sample workflow: every sub-threshold peak identified in the baseline-corrected spectrum is confirmed to be absent from the peak-filtered output
- Each retained peak's intensity divided by its segment MAD satisfies the user-specified SNR_thresh
- Slope-based endpoint filtering has removed flagged candidates that lack the characteristic shape of real peaks
- Output remains a data frame with m/z and intensity data accessible (no S3/S4 masking) after peak detection
- Peak-filtered spectrum is suitable for the downstream steps of alignment to the m/z grid at specified resolution and normalization

## Limitations

- The article does not name the specific peak detection algorithms included in the package, so algorithm selection must be resolved from the package's vignettes/source
- No software version numbers are stated for subMALDI, R, or dependencies, and no changelog/version history is provided
- Installing from source requires the BioConductor dependency MassSpecWavelet, and the README notes the BioConductor install sequence 'doesn't always seem to work'; installing with dependencies = TRUE is the workaround
- No DOI or data availability statement is given for the datasets used in the package's workflow

## Evidence

- [abstract] subMALDI permits SNR-based peak detection on unmasked data frames: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets"
- [abstract] subMALDI's open, unmasked data design keeps m/z and intensity accessible after each processing step: "permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects"
- [abstract] Peak-filtered output feeds into m/z-grid alignment at a specified resolution: "aligned to an array of all the theoretically possible m/z values in the observed mass range, at a specified resolution"
- [intro] subMALDI data objects are compatible with other R packages, enabling user-defined peak detection functions: "subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] README directs users to the vignettes for workflow and processing examples: [evidence span withheld — anchor status: external_doc]
- [readme] README documents the MassSpecWavelet BioConductor dependency required for installation: "a dependency from the BioConductor project is required. Begin by installing the BioConductor Manager: install.packages("BiocManager") and then install the package MassSpecWavelet"
