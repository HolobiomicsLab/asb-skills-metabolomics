---
name: out-of-memory-spectral-data-processing
description: Use when raw MSI spectra (e.g. imzML or Analyze 7.5 files) are too large
  to fit in memory, and pre-processing (normalize, smooth, reduceBaseline, peakPick)
  or PCA must be run on the full dataset.
when_to_use_negative:
- Input is already a small peak-picked or feature table that fits comfortably in memory
  — streaming overhead offers no benefit
- Data is not MS imaging spectra (e.g. general genomic tables); matter/Cardinal is
  specialized for MSI experiments
- Using the unstable GitHub devel or Bioconductor devel version for critical work
  — README explicitly warns these should not be used for critical work
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Cardinal
  role: R package providing MSI import (readMSIData), queued pre-processing, and statistical
    analysis on larger-than-memory data
  repo: github.com/kuwisdelu/Cardinal
- name: matter
  role: Backend enabling Cardinal's computing on larger-than-memory MS imaging datasets
- name: BiocParallel
  role: Parallel processing support for pre-processing methods
- name: BiocManager
  role: Installer for Cardinal via Bioconductor
- name: R
  role: Runtime environment in which Cardinal and matter execute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:bd2aee0933dceb16f0639fc17a71c2b4d4ddf50caa13b6debe362f97780f5593
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# out-of-memory-spectral-data-processing

## Summary

Process MS imaging datasets larger than available RAM using Cardinal's matter backend, which streams spectral data from disk so raw imzML/Analyze 7.5 spectra need not be fully loaded into memory during pre-processing or statistical analysis.

## When to use

Use when raw MSI spectra (e.g. imzML or Analyze 7.5 files) are too large to fit in memory, and pre-processing (normalize, smooth, reduceBaseline, peakPick) or PCA must be run on the full dataset.

## When NOT to use

- Input is already a small peak-picked or feature table that fits comfortably in memory — streaming overhead offers no benefit
- Data is not MS imaging spectra (e.g. general genomic tables); matter/Cardinal is specialized for MSI experiments
- Using the unstable GitHub devel or Bioconductor devel version for critical work — README explicitly warns these should not be used for critical work

## Inputs

- Raw (unprocessed) MS imaging spectra in imzML ('continuous' or 'processed') or Analyze 7.5 format
- Dataset whose spectra exceed available system memory
- Optional peak-picked spectra for fast downstream statistics

## Outputs

- Pre-processed MS imaging experiment (normalized, smoothed, baseline-reduced, peak-picked) computed on-disk without full in-memory loading
- PCA components / statistical analysis results computed on larger-than-memory data
- Timing record (e.g. pca_timing.csv) with elapsed time, method (PCA, ncomp=20), and dataset name

## How to apply

Install Cardinal via BiocManager and import the larger-than-memory dataset with readMSIData(), which natively reads both 'continuous' and 'processed' imzML and Analyze 7.5 formats. Cardinal computes on top of the matter package backend, which was rewritten in Cardinal 3 to provide more robust support for datasets whose raw spectra cannot be fully loaded into memory. Queue spectral processing steps (normalize, smooth, reduceBaseline, recalibrate, peakPick/peakAlign) and apply them in sequence with process(), so operations are applied to mass spectra individually and streamed rather than materializing the full array in RAM. Optionally use bin() to bin or resample mass spectra, and enable parallel pre-processing via the BiocParallel package. After processing, run statistical methods such as PCA on the on-disk representation; on raw pig fetus data this design let Cardinal compute the first 20 principal components in 86.9 s (4.3 s on picked peaks) on a 2.6 GHz Intel Core i7 MacBook Pro with 16 GB memory.

## Related tools

- **Cardinal** (R package providing MSI import (readMSIData), queued pre-processing, and statistical analysis on larger-than-memory data) — github.com/kuwisdelu/Cardinal
- **matter** (Backend enabling Cardinal's computing on larger-than-memory MS imaging datasets)
- **BiocParallel** (Parallel processing support for pre-processing methods)
- **BiocManager** (Installer for Cardinal via Bioconductor)
- **R** (Runtime environment in which Cardinal and matter execute)

## Examples

```
BiocManager::install("Cardinal"); library(Cardinal); msi <- readMSIData("pig_fetus.imzML"); pca <- PCA(msi, ncomp=20)
```

## Evaluation signals

- Elapsed time for PCA (first 20 components) on raw pig fetus data is of comparable magnitude to the reported 86.9 s on a 2.6 GHz Intel Core i7 / 16 GB MacBook Pro (hardware-dependent; picked-peak timing was 4.3 s)
- Computation completes on a raw dataset larger than RAM without out-of-memory errors, confirming matter backend streaming is active
- Queued processing steps (normalize, smooth, reduceBaseline, peakPick) are all applied only after calling process()
- Imported object correctly represents the imzML or Analyze 7.5 file, including 'continuous' vs 'processed' imzML variants

## Limitations

- Absolute runtimes are hardware-dependent — the 86.9 s figure applies to a specific 2.6 GHz MacBook Pro and must be treated as expert judgment, not a portable benchmark
- Legacy support has been dropped in Cardinal 3 (requires matter 2), so older scripts may break
- Cardinal 3.6 introduces breaking changes; many redundant functions were merged and unnecessary functions deprecated
- GitHub devel and Bioconductor devel versions are unstable and should not be used for critical work

## Evidence

- [intro] matter backend supports larger-than-memory: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten"
- [intro] raw spectra exceed memory assumption: "the basic assumption that the raw spectra can be fully loaded into memory is unreasonable for many MS imaging experiments today."
- [intro] supported file formats and import: "*Cardinal* natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats via the `readMSIData()` and `writeMSIData()` functions."
- [intro] queued processing model: "Processing steps that are applied to mass spectra individually are be queued to be applied in sequence later. Use `process()` to apply queued processing steps."
- [readme] PCA and PLS among core methods: [evidence span withheld — anchor status: external_doc]
- [intro] parallel pre-processing: "Parallel processing support via the *BiocParallel* package for all pre-processing methods"
