---
name: msi-principal-component-analysis
description: Use when the input is an MSI dataset (imzML or Analyze 7.5) and the goal
  is unsupervised exploratory analysis to discover regions of interest, or when benchmarking
  the runtime cost of PCA on raw versus pre-processed/peak-picked spectra.
when_to_use_negative:
- The goal is supervised image classification or segmentation with predefined classes
  — Cardinal offers dedicated classification/segmentation methods based on nearest
  shrunken centroids and mixture modeling instead.
- Spectra have not undergone any pre-processing but a fully processed feature matrix
  is required downstream and peak picking has not yet been run — apply peakPick/peakProcess
  first for the runtime benefit.
- The data are not mass spectrometry imaging spectra (e.g. already a generic feature
  table unrelated to MSI).
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Cardinal
  role: R package providing the PCA() function and MSI data handling (imzML/Analyze
    7.5 import, pre-processing, plotting) used to run principal component analysis
    on MSI spectra
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment in which Cardinal is installed via BiocManager and where
    timing (system.time) is applied around PCA calls
- name: matter
  role: Backend enabling Cardinal's computing on larger-than-memory MS imaging datasets
    so PCA can run on raw spectra that cannot be fully loaded into memory
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:c0a1e7ca9fead578e65bb8252a9f282f17121f03fbcbfccf82d02b8667a1a289
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# msi-principal-component-analysis

## Summary

Runs PCA on mass spectrometry imaging (MSI) spectra using Cardinal in R, computing the first N principal components (e.g. ncomp=20) on either raw or peak-picked spectra. Peak picking before PCA yields large speedups (reported ~4.3 s on picked peaks vs ~86.9 s raw for 20 components).

## When to use

Use when the input is an MSI dataset (imzML or Analyze 7.5) and the goal is unsupervised exploratory analysis to discover regions of interest, or when benchmarking the runtime cost of PCA on raw versus pre-processed/peak-picked spectra.

## When NOT to use

- The goal is supervised image classification or segmentation with predefined classes — Cardinal offers dedicated classification/segmentation methods based on nearest shrunken centroids and mixture modeling instead.
- Spectra have not undergone any pre-processing but a fully processed feature matrix is required downstream and peak picking has not yet been run — apply peakPick/peakProcess first for the runtime benefit.
- The data are not mass spectrometry imaging spectra (e.g. already a generic feature table unrelated to MSI).

## Inputs

- MS imaging spectra in imzML ('continuous' or 'processed') or Analyze 7.5 format, readable via Cardinal's readMSIData()
- Raw (unprocessed) pig fetus MSI spectra, or peak-picked spectra from an upstream peakPick/peakProcess step
- Number of principal components to compute (e.g. ncomp=20)

## Outputs

- First 20 (or user-specified) principal components / PCA scores from Cardinal's PCA()
- Measured elapsed computation time (e.g. from system.time()) written to a structured timing record such as pca_timing.csv with method, component count, and data state

## How to apply

Load the MSI spectra into R with Cardinal (raw imzML can be read via readMSIData(), including 'continuous' and 'processed' imzML types). Decide whether to run PCA on the raw spectra or on peak-picked data produced with peakPick()/peakAlign()/peakProcess(); peak picking dramatically reduces runtime because PCA then operates on summarized peaks rather than full spectra. Call PCA() requesting the desired number of components (e.g. the first 20), wrapping the call in system.time() to measure elapsed computation. Cardinal's matter backend supports larger-than-memory MS imaging datasets, so raw spectra need not fit fully in memory. Write elapsed time, method (PCA, ncomp), data state (raw vs peak-picked), and dataset name into a structured record such as pca_timing.csv.

## Related tools

- **Cardinal** (R package providing the PCA() function and MSI data handling (imzML/Analyze 7.5 import, pre-processing, plotting) used to run principal component analysis on MSI spectra) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment in which Cardinal is installed via BiocManager and where timing (system.time) is applied around PCA calls)
- **matter** (Backend enabling Cardinal's computing on larger-than-memory MS imaging datasets so PCA can run on raw spectra that cannot be fully loaded into memory)

## Examples

```
system.time(p <- PCA(msiData, ncomp=20))
```

## Evaluation signals

- Recorded elapsed time in pca_timing.csv exists and includes method (PCA), component count (ncomp=20), and data state (raw vs peak-picked).
- Timing on peak-picked spectra is on the order of seconds (reported ~4.3 s), substantially lower than raw-spectra PCA (~86.9 s), demonstrating the peak-picking speedup.
- Absolute runtime is of comparable magnitude to the paper's 86.9 s reference, modulo hardware differences (reference: 2.6 GHz Intel Core i7 MacBook Pro, 16 GB memory) — treat absolute runtime as hardware-dependent expert judgment.
- PCA is requested with the same component count (20) as the benchmark so timing comparisons are apples-to-apples.

## Limitations

- Runtimes are hardware-dependent: the 86.9 s / 4.3 s figures were reported on a MacBook Pro with a 2.6 GHz Intel Core i7 and 16 GB memory, so absolute timings on other machines will differ.
- Legacy support was dropped in Cardinal 3 (updated for matter 2), so older script versions may not run; Cardinal 3.6 introduced breaking changes and deprecated many redundant functions.
- PCA on raw spectra is far more expensive because raw spectra may not fit in memory for many modern MS imaging experiments; the larger-than-memory path relies on the rewritten matter backend.

## Evidence

- [abstract] Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues.: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [readme] PCA and PLS in README: [evidence span withheld — anchor status: external_doc]
- [intro] matter larger-than-memory backend: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten."
- [intro] peakPick for peak detection: "`peakPick()` for peak detection and summarization"
- [intro] imzML import via readMSIData: "*Cardinal* natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats via the `readMSIData()` and `writeMSIData()` functions."
