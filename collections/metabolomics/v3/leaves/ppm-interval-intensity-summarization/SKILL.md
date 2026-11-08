---
name: ppm-interval-intensity-summarization
description: Use when you have a matrix of aligned, baseline-corrected 1D 1H NMR spectra
  (ppm scale x samples) from metabolic fingerprinting and need to reduce them to a
  samples x buckets intensity table while keeping peaks with uncontrolled chemical-shift
  variations inside single buckets.
when_to_use_negative:
- The input is already a bucket/feature table — this skill operates on raw aligned
  spectral matrices, not on pre-binned intensities
- Targeted metabolomics requiring per-resonance quantification of crowded, heavily
  overlapped regions — NMRProcFlow's local signal deconvolution quantification method
  is only planned in a future version and not available here
- Spectra have not yet undergone ppm-scale calibration, baseline correction and realignment
  — bucketing shifted, uncorrected spectra would reintroduce the peak-shift artifacts
  this skill exists to avoid
tools:
- name: NMRProcFlow
  role: GUI pipeline whose Rnmr1D module runs the Adaptive Intelligent Binning algorithm
    and produces the bucket table
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: R-based spectra processing application that prepares the aligned, baseline-corrected
    spectra fed into binning
- name: R
  role: implementation language of the spectra processing and binning algorithms in
    Rnmr1D
- name: Rcpp
  role: package embedding the C++ binning algorithms into R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:cdbbcd53cf97fab0444d8db3da4b205bb531d32e5112e3c0b0d3a88cceffe3f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - R
  - Rcpp
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ppm-interval-intensity-summarization

## Summary

Summarizes aligned, baseline-corrected 1D 1H NMR spectra into a bucket table by summing intensities over ppm intervals, using the Adaptive Intelligent Binning algorithm (De Meyer et al. 2008) to avoid the information loss and peak-shift artifacts of equally sized bins.

## When to use

Use when you have a matrix of aligned, baseline-corrected 1D 1H NMR spectra (ppm scale x samples) from metabolic fingerprinting and need to reduce them to a samples x buckets intensity table while keeping peaks with uncontrolled chemical-shift variations inside single buckets.

## When NOT to use

- The input is already a bucket/feature table — this skill operates on raw aligned spectral matrices, not on pre-binned intensities
- Targeted metabolomics requiring per-resonance quantification of crowded, heavily overlapped regions — NMRProcFlow's local signal deconvolution quantification method is only planned in a future version and not available here
- Spectra have not yet undergone ppm-scale calibration, baseline correction and realignment — bucketing shifted, uncorrected spectra would reintroduce the peak-shift artifacts this skill exists to avoid

## Inputs

- matrix of aligned, baseline-corrected 1D 1H NMR spectra (ppm scale x samples), produced with the NMRProcFlow/NMRspec processing pipeline
- optional signal-to-noise ratio filter configuration for bucket restriction at output

## Outputs

- buckets.tsv: tab-separated bucket table with spectral intervals (ppm boundaries) as row identifiers and one non-negative intensity column per sample

## How to apply

Load the matrix of aligned, baseline-corrected spectra into NMRProcFlow's Rnmr1D module (R with C++ algorithms via Rcpp) and invoke the Intelligent binning method rather than fixed-width or variable-size binning: the Adaptive Intelligent Binning algorithm recursively identifies new bin edges inside existing bins so that resonances whose chemical shifts drift between spectra stay within one bucket, requiring minimal user input and no arbitrary parameters or reference spectra. Sum spectral intensities over each adaptive interval for every sample to build the bucket table (rows = spectral intervals in ppm, columns = samples). Optionally apply the signal-to-noise bucket filter either during the computing process or at the output step to restrict buckets. Write the result to buckets.tsv and validate that it is a well-formed tab-separated table with one intensity column per input sample and complete rows of non-negative intensities.

## Related tools

- **NMRProcFlow** (GUI pipeline whose Rnmr1D module runs the Adaptive Intelligent Binning algorithm and produces the bucket table) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (R-based spectra processing application that prepares the aligned, baseline-corrected spectra fed into binning)
- **R** (implementation language of the spectra processing and binning algorithms in Rnmr1D)
- **Rcpp** (package embedding the C++ binning algorithms into R)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- buckets.tsv exists and is a well-formed tab-separated table with one intensity column per input sample
- row identifiers are interval boundaries in ppm, covering the analyzed spectral range
- every sample yields a complete row of non-negative bucket intensities (no missing values)
- peaks with uncontrolled chemical-shift variations between spectra fall within a single bucket rather than being split across bin edges
- buckets retained after the optional signal-to-noise filter are consistent with the chosen filtering mode (during computing or at output)

## Limitations

- Equally sized binning — the fallback mode — loses information and creates peak-shift artifacts; only the Intelligent binning mode avoids them
- Spectra alignment, which precedes binning, remains a challenge requiring user expertise; automated batch-mode processing is not fully mature
- Bucketing produces interval-summed intensities, not individual peak quantification; local signal deconvolution for crowded regions is planned only for a future version
- NMRProcFlow deployment constrains working sessions (PURGESESSIONS=2 days before session destruction, MAXZIPSIZE=400 MB)

## Evidence

- [results] Adaptive Intelligent Binning avoids equally sized bin problems: "Therefore, we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges in existing bins."
- [results] SNR bucket filter can run during computing or at output: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [readme] Binning is one of the processing steps in NMRProcFlow: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or"
