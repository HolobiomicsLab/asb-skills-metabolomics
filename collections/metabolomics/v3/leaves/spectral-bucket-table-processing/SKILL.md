---
name: spectral-bucket-table-processing
description: Use when you have a matrix of processed 1D 1H NMR spectra (ppm scale
  x samples) from NMRProcFlow/NMRspec and need to reduce them to a bucket table with
  adaptive intervals and SNR-based quality filtering.
when_to_use_negative:
- Input spectra are raw FIDs that have not yet gone through baseline correction, chemical
  shift calibration and alignment in NMRProcFlow
- The analysis needs targeted peak quantification rather than metabolic-fingerprinting
  bucketing (NMRProcFlow plans local signal deconvolution for crowded regions only
  in a future version)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: GUI tool that exposes binning (Intelligent, variable size or fixed width)
    and the SNR bucket filter either during computing or at output
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: R-based spectra processing application that prepares the aligned, baseline-corrected
    spectra fed to the bucketing step
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Implementation language of NMRspec/Rnmr1D in which the Intelligent binning
    and bucket-table routines run
- name: Rcpp
  role: Package used to embed the C++ implementations of processing algorithms (e.g.
    binning) callable from R
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:e5df32b796ee1a22e3d9b1a39b8b0300b9be9623c6ebaf755ce86e0ea68da82f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-bucket-table-processing

## Summary

Builds and filters a bucket table from aligned, baseline-corrected 1D 1H NMR spectra using NMRProcFlow's Adaptive Intelligent Binning (De Meyer et al. 2008) plus an optional signal-to-noise ratio (SNR) filter, so that buckets fed to statistics are information-preserving and noise-free.

## When to use

Use when you have a matrix of processed 1D 1H NMR spectra (ppm scale x samples) from NMRProcFlow/NMRspec and need to reduce them to a bucket table with adaptive intervals and SNR-based quality filtering.

## When NOT to use

- Input spectra are raw FIDs that have not yet gone through baseline correction, chemical shift calibration and alignment in NMRProcFlow
- The analysis needs targeted peak quantification rather than metabolic-fingerprinting bucketing (NMRProcFlow plans local signal deconvolution for crowded regions only in a future version)

## Inputs

- Matrix of aligned, baseline-corrected 1D 1H NMR spectra (ppm scale x samples) from the NMRProcFlow/NMRspec processing pipeline
- Chosen binning mode (Intelligent, variable size, or fixed width)
- SNR threshold for bucket retention/discarding

## Outputs

- Bucket table with rows = spectral intervals in ppm and columns = samples of bucket intensities (e.g. buckets.tsv)
- SNR-filtered bucket table (e.g. filtered_bucket_table.csv) retaining only buckets whose SNR exceeds the threshold

## How to apply

Load the aligned, baseline-corrected 1D 1H NMR spectra (ppm scale x samples) prepared with NMRProcFlow/NMRspec and invoke the Rnmr1D module (R with C++ algorithms via Rcpp) to run the Intelligent binning method, which recursively identifies bin edges within existing bins so peaks with uncontrolled chemical-shift variations stay within one bucket — avoiding the loss of information and peak-shift artifacts of equally sized bins. Alternatively choose variable-size or fixed-width bins as exposed by NMRProcFlow's binning step. Sum the spectral intensities over each adaptive interval for every sample to build the bucket table (rows = ppm intervals, columns = samples). Apply the SNR filter: compute the signal-to-noise ratio for each bucket and either discard buckets below the chosen threshold during the computing process or at the output step. Write the retained buckets with their ppm ranges and intensities to a table file (e.g. buckets.tsv or filtered_bucket_table.csv) and validate that row counts match the buckets passing the SNR threshold.

## Related tools

- **NMRProcFlow** (GUI tool that exposes binning (Intelligent, variable size or fixed width) and the SNR bucket filter either during computing or at output) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (R-based spectra processing application that prepares the aligned, baseline-corrected spectra fed to the bucketing step) — https://github.com/INRA/NMRProcFlow
- **R** (Implementation language of NMRspec/Rnmr1D in which the Intelligent binning and bucket-table routines run)
- **Rcpp** (Package used to embed the C++ implementations of processing algorithms (e.g. binning) callable from R)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- buckets.tsv exists and is a well-formed tab-separated table with one intensity column per input sample and interval boundaries in ppm as row identifiers
- Every sample yields a complete row of non-negative bucket intensities
- filtered_bucket_table.csv is a valid CSV whose row count equals the number of buckets whose SNR exceeds the threshold
- Peaks with uncontrolled chemical-shift variations between spectra fall within a single bucket (no peak splitting across bins)
- Every retained bucket carries an explicit ppm range identifier and per-sample intensity values

## Limitations

- Bucketing is for metabolic fingerprinting; a quantification method based on local signal deconvolution for crowded regions with heavy peak overlap is only planned in a future version of NMRProcFlow
- Spectra alignment upstream requires user expertise and remains an unsolved challenge in 1D NMR spectra processing
- Buckets can only be filtered by SNR (during computing or at output); other quality filters are not described

## Evidence

- [results] Adaptive Intelligent Binning implemented for bucketing: "Therefore, we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges in existing bins."
- [results] SNR filter applied during computing or at output: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [methods] Binning options in spectra processing: "the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
- [methods] C++ algorithms via Rcpp: "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
- [abstract] Upstream processing steps required before bucketing: "covers all spectra processing steps including baseline correction, chemical shift calibration and alignment."
