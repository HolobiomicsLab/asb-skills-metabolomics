---
name: 1d-nmr-metabolomics-noise-thresholding
description: Use when a bucket table of ppm intervals with per-spectrum intensities
  has been produced by binning (Intelligent, variable size or fixed width) and noisy
  buckets must be retained or discarded based on SNR before statistical analysis.
when_to_use_negative:
- Input is already a cleaned/filtered feature table with no raw bucket intensities
  available for SNR computation
- The task concerns upstream spectra processing steps (ppm scale calibration, baseline
  correction, realignment) rather than post-binning bucket filtering
- Quantification of crowded regions with heavy peak overlap is needed — the article
  states a local signal deconvolution method for this was only planned in a future
  version
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: GUI tool that performs the binning and provides the option to filter buckets
    based on signal-to-noise ratio either during the computing process or at the output
    step
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Open source language in which NMRProcFlow's spectra processing (NMRspec) is
    implemented; usable to script SNR computation and thresholding on the bucket table
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: Container image (docker.io/nmrprocflow/npflow) used to deploy the NMRProcFlow
    application that carries out the bucketing and SNR filtering
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:a687ca65918879da76fa80b420ad22c18697ebd1acb5a83d66a34d77870ec1aa
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# 1d-nmr-metabolomics-noise-thresholding

## Summary

Filter buckets of a 1D NMR bucket table based on their signal-to-noise ratio (SNR), as implemented in NMRProcFlow, so that buckets dominated by noise are discarded and only signal-bearing ppm intervals are retained for downstream statistics.

## When to use

Use when a bucket table of ppm intervals with per-spectrum intensities has been produced by binning (Intelligent, variable size or fixed width) and noisy buckets must be retained or discarded based on SNR before statistical analysis.

## When NOT to use

- Input is already a cleaned/filtered feature table with no raw bucket intensities available for SNR computation
- The task concerns upstream spectra processing steps (ppm scale calibration, baseline correction, realignment) rather than post-binning bucket filtering
- Quantification of crowded regions with heavy peak overlap is needed — the article states a local signal deconvolution method for this was only planned in a future version

## Inputs

- Bucket table of ppm intervals with per-spectrum intensities produced by NMRProcFlow binning (Intelligent/Adaptive Intelligent Binning, variable size or fixed width)
- 1D NMR spectra (1H & 13C) processed in NMRProcFlow, e.g. from Bruker or Agilent (Varian) FID files
- User-chosen SNR threshold for bucket retention

## Outputs

- Filtered bucket table containing only buckets whose SNR exceeds the threshold, with ppm ranges and intensities (e.g. filtered_bucket_table.csv)

## How to apply

Load the bucket table produced by the binning stage of NMRProcFlow (Adaptive Intelligent Binning, variable-size or fixed-width bins). For each bucket, compute the signal-to-noise ratio from its intensity values relative to the spectral noise level. NMRProcFlow allows the SNR filter to be applied in two ways: 'either during the computing process, or at the output step', so choose the point of application that fits the workflow. Discard every bucket whose SNR falls below the chosen threshold and retain the rest, then export the retained buckets with their ppm ranges and intensities. The rationale is a quality-based mechanism: buckets dominated by noise carry no metabolite information and would degrade downstream multivariate analyses.

## Related tools

- **NMRProcFlow** (GUI tool that performs the binning and provides the option to filter buckets based on signal-to-noise ratio either during the computing process or at the output step) — https://github.com/INRA/NMRProcFlow
- **R** (Open source language in which NMRProcFlow's spectra processing (NMRspec) is implemented; usable to script SNR computation and thresholding on the bucket table) — https://github.com/INRA/NMRProcFlow
- **Docker** (Container image (docker.io/nmrprocflow/npflow) used to deploy the NMRProcFlow application that carries out the bucketing and SNR filtering) — https://github.com/INRA/NMRProcFlow

## Examples

```
$ docker pull docker.io/nmrprocflow/npflow && sh ./npflow start  # then apply the SNR bucket filter in the NMRProcFlow GUI at the output step
```

## Evaluation signals

- The output file (e.g. filtered_bucket_table.csv) exists and is a valid CSV
- Row count of the output equals the number of buckets whose computed SNR exceeds the chosen threshold
- Every retained bucket's SNR is at or above the threshold and every discarded bucket's SNR is below it
- Retained buckets preserve their original ppm ranges and per-spectrum intensities from the bucket table
- The filtering stage follows the binning step (Intelligent, variable size or fixed width) and precedes statistical analysis of the bucket table

## Limitations

- The article does not specify a default SNR threshold value, so the cutoff must be chosen by the user
- For crowded regions with heavy peak overlap, SNR filtering alone is insufficient — a quantification method based on local signal deconvolution was only 'planned in a future version of NMRProcFlow'
- Applying the filter during computing versus at the output step may yield different intermediate results; the choice of application point is left to the user

## Evidence

- [results] SNR filter applied at output: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [methods] binning step context: "and the binning (Intelligent, variable size or fixed width)"
- [results] Adaptive Intelligent Binning implementation: "we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges"
- [readme] processing scope of the tool: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning"
