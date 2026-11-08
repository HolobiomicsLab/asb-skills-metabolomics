---
name: nmr-bucket-snr-filtering
description: Use when a 1D 1H/13C NMR spectra processing pipeline has produced a bucket
  table (ppm intervals × per-spectrum intensities) and noisy, low-intensity buckets
  must be removed before statistical analysis.
when_to_use_negative:
- Input is raw FID or unprocessed spectra — binning must be done first so a bucket
  table exists to filter
- The analysis goal is peak deconvolution or quantification of crowded, heavily overlapping
  regions, which NMRProcFlow planned as a future feature rather than SNR-based bucket
  filtering
- Only baseline correction, ppm calibration, or alignment is needed — these are separate
  upstream processing steps, not bucket filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRProcFlow
  role: Provides the GUI and processing pipeline in which buckets can be filtered
    based on signal-to-noise ratio either during the computing process or at the output
    step
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Implementation language of the NMRspec spectra processing application underlying
    NMRProcFlow, usable to script SNR computation and bucket filtering on the bucket
    table
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: 'Deployment vehicle: the npflow image embeds the application modules sharing
    a data volume and communicating via AJAX web services'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:903bd85d136c2cb86e63504d07a02e98944a32aac71b730bf1c1cdbe9b6d9db3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmr-bucket-snr-filtering

## Summary

Retain or discard buckets from an NMR bucket table according to their signal-to-noise ratio (SNR), as implemented in NMRProcFlow, which permits this filtering either during the computing process or at the output step.

## When to use

Use when a 1D 1H/13C NMR spectra processing pipeline has produced a bucket table (ppm intervals × per-spectrum intensities) and noisy, low-intensity buckets must be removed before statistical analysis.

## When NOT to use

- Input is raw FID or unprocessed spectra — binning must be done first so a bucket table exists to filter
- The analysis goal is peak deconvolution or quantification of crowded, heavily overlapping regions, which NMRProcFlow planned as a future feature rather than SNR-based bucket filtering
- Only baseline correction, ppm calibration, or alignment is needed — these are separate upstream processing steps, not bucket filtering

## Inputs

- Bucket table of ppm intervals with per-spectrum intensities produced by NMRProcFlow binning (Intelligent, variable size or fixed width)
- Processed 1D NMR spectra (1H or 13C) from Bruker or Agilent (Varian) FID files, or nmrML format
- User-chosen SNR threshold

## Outputs

- Filtered bucket table retaining only buckets whose SNR exceeds the threshold
- Discarded bucket list (buckets below the SNR cutoff)

## How to apply

Load the bucket table produced by the binning stage (Adaptive Intelligent Binning, variable-size or fixed-width bins) in NMRProcFlow. Compute the signal-to-noise ratio for each bucket from its intensities relative to the spectral noise level. Choose whether to apply the SNR filter during the computing process or at the output step — NMRProcFlow supports both points. Discard every bucket whose SNR falls below the chosen threshold and retain the rest; the retained buckets, with their ppm ranges and intensities, form the cleaned bucket table for downstream statistics. The rationale is a quality-based removal of bins carrying more noise than signal, complementing the upstream ppm-scale calibration, baseline correction, realignment, and binning steps.

## Related tools

- **NMRProcFlow** (Provides the GUI and processing pipeline in which buckets can be filtered based on signal-to-noise ratio either during the computing process or at the output step) — https://github.com/INRA/NMRProcFlow
- **R** (Implementation language of the NMRspec spectra processing application underlying NMRProcFlow, usable to script SNR computation and bucket filtering on the bucket table) — https://github.com/INRA/NMRProcFlow
- **Docker** (Deployment vehicle: the npflow image embeds the application modules sharing a data volume and communicating via AJAX web services)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git; docker pull docker.io/nmrprocflow/npflow; sh ./npflow start; then check http://<your_local_host>:<port>/npflow/
```

## Evaluation signals

- The output filtered bucket table exists, is a valid CSV, and its row count equals the number of buckets whose SNR exceeds the threshold
- Every retained bucket's SNR is above the chosen threshold; every discarded bucket's SNR is below it
- Retained buckets keep their original ppm ranges and per-spectrum intensities unchanged by the filtering step
- The number of spectra (columns) is identical before and after filtering — only bucket rows are removed
- Filtering was applied at one of the two supported points: during the computing process or at the output step

## Limitations

- No default SNR threshold value is specified in the article; the cutoff is user-chosen and depends on the spectral noise level
- Bucket filtering cannot recover information lost by poor binning — the Adaptive Intelligent Binning step must already have minimized peak-shift artifacts
- SNR-based filtering does not address peak overlap in crowded regions; a local signal deconvolution quantification method was only planned for a future NMRProcFlow version

## Evidence

- [results] SNR filter point (results): "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [readme] Processing steps context (README): "the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
- [abstract] Full processing scope (abstract): "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction"
