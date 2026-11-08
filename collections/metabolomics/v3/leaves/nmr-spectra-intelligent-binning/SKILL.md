---
name: nmr-spectra-intelligent-binning
description: Use when bucketing aligned, baseline-corrected 1D 1H NMR spectra into
  a sample x bucket intensity matrix and peak-shift artifacts or information loss
  from equally sized bins are a concern.
when_to_use_negative:
- Input is already a bucketed or feature-level data matrix rather than processed 1D
  spectra on the ppm scale
- A targeted metabolomics quantification is required where analyte-specific peak fitting
  (e.g. planned local signal deconvolution for crowded regions) is needed rather than
  fingerprinting-style bucketing
- Equally sized fixed-width bins are explicitly required by the downstream analysis
  and peak-shift artifacts are not a concern
edam_operation: http://edamontology.org/operation_0336
tools:
- name: NMRProcFlow
  role: GUI application (run from the 'npflow' Docker image) whose Rnmr1D module executes
    Intelligent binning on the aligned spectra and exports the bucket table
  repo: https://github.com/INRA/NMRProcFlow
- name: Rcpp
  role: R package used to implement the C++ processing algorithms, including the recursive
    bin-edge identification of Adaptive Intelligent Binning
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:43975ca9a96f8dd31f68659bc2a76a3cb96868f33f386b2e4c298ee5a6671f32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - NMRspec
  - R
  - R Shiny
  - Rcpp
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nmr-spectra-intelligent-binning

## Summary

Adaptive Intelligent Binning (De Meyer et al. 2008) as implemented in NMRProcFlow's Rnmr1D module: it buckets aligned 1D 1H NMR spectra by recursively identifying bin edges within existing bins, avoiding the loss of information and peak-shift artifacts caused by equally sized bins.

## When to use

Use when bucketing aligned, baseline-corrected 1D 1H NMR spectra into a sample x bucket intensity matrix and peak-shift artifacts or information loss from equally sized bins are a concern.

## When NOT to use

- Input is already a bucketed or feature-level data matrix rather than processed 1D spectra on the ppm scale
- A targeted metabolomics quantification is required where analyte-specific peak fitting (e.g. planned local signal deconvolution for crowded regions) is needed rather than fingerprinting-style bucketing
- Equally sized fixed-width bins are explicitly required by the downstream analysis and peak-shift artifacts are not a concern

## Inputs

- Matrix of aligned, baseline-corrected 1D 1H NMR spectra on the ppm scale (rows = ppm points, columns = samples)
- Bin/interval definition for Intelligent binning (user-selected method in the NMRProcFlow GUI; minimal user input)
- Optional signal-to-noise ratio filter setting applied during computing or at output

## Outputs

- Bucket table: rows = adaptive spectral intervals in ppm, columns = samples, values = summed non-negative bucket intensities (e.g. buckets.tsv or CSV)
- Buckets passing the signal-to-noise filter (excluded buckets are removed from the output)

## How to apply

Start from the matrix of aligned, baseline-corrected 1D 1H NMR spectra (ppm scale x samples) produced by the NMRProcFlow/NMRspec pipeline, after ppm-scale calibration, local or full baseline correction, and removal of solvent/contaminant resonance zones. Invoke the Rnmr1D module (R with C++ algorithms via the Rcpp package) and select the Intelligent binning method rather than fixed-width or variable-size binning; the algorithm recursively identifies bin edges within existing bins so that peaks with uncontrolled chemical-shift variations between spectra stay within a single bucket, requiring minimal user input and no arbitrary parameters or reference spectra. Sum spectral intensities over each adaptive interval for every sample to build the bucket table (rows = ppm intervals, columns = samples). Optionally filter buckets by signal-to-noise ratio either during the computing process or at the output step. Export the bucket table (e.g. buckets.tsv / CSV) as the final data matrix.

## Related tools

- **NMRProcFlow** (GUI application (run from the 'npflow' Docker image) whose Rnmr1D module executes Intelligent binning on the aligned spectra and exports the bucket table) — https://github.com/INRA/NMRProcFlow
- **Rcpp** (R package used to implement the C++ processing algorithms, including the recursive bin-edge identification of Adaptive Intelligent Binning) — https://github.com/INRA/NMRProcFlow

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start  # then select Intelligent binning in the NMRProcFlow GUI
```

## Evaluation signals

- buckets.tsv exists and is a well-formed tab-separated table with one intensity column per input sample
- Row identifiers are interval boundaries in ppm on the spectral scale
- Every sample yields a complete row of non-negative bucket intensities
- Peaks with uncontrolled chemical-shift variations across spectra fall within single buckets rather than being split across bin boundaries
- Buckets failing the signal-to-noise ratio filter are excluded from the exported matrix

## Limitations

- Intelligent binning is one of several binning options (Intelligent, variable size, or fixed width); it must be explicitly selected over the others
- The article notes that spectra alignment, which precedes binning, still requires user expertise
- Crowded regions with heavy peak overlap need a quantification method based on local signal deconvolution, planned for a future version, which binning alone does not cover

## Evidence

- [results] Therefore, we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges in existing bins.: "Therefore, we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges in existing bins."
- [readme] the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width): "the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
- [results] in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step.: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [methods] some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011): "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
