---
name: runtime-benchmarking-recording
description: Use when a research question asks how long a named tool/operation (e.g.
  PCA on raw MSI spectra) takes to run, and the answer must be a measured, reproducible
  timing record tied to the hardware it ran on.
when_to_use_negative:
- When the goal is to assess statistical quality (e.g. segmentation accuracy, classification
  error) rather than computational cost — Cardinal's statistical methods are the package's
  core functionality and need different evaluation.
- When validating an exact numerical reproduction is expected — runtime is hardware-dependent,
  so it cannot be matched to a reference number precisely.
- When the input is already a timing/performance table — the skill measures and records
  runtimes, it does not re-analyze existing benchmark data.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Cardinal
  role: R package in which the timed operation (PCA, ncomp=20) is executed on the
    MSI spectra
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment used to load data, time the call (Sys.time), and write
    the timing CSV
- name: BiocManager
  role: Used to install Cardinal via BiocManager::install("Cardinal") before benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:5050197c8b28117415be0be9af7308c4d9c30f9fe4d0ef37e59cbee79a4e3b5c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Cardinal
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# runtime-benchmarking-recording

## Summary

Benchmark the wall-clock runtime of a specific computational operation — here, PCA on raw mass spectrometry imaging spectra in Cardinal — and record the elapsed time with method, dataset, and hardware context in a structured timing file.

## When to use

Use when a research question asks how long a named tool/operation (e.g. PCA on raw MSI spectra) takes to run, and the answer must be a measured, reproducible timing record tied to the hardware it ran on.

## When NOT to use

- When the goal is to assess statistical quality (e.g. segmentation accuracy, classification error) rather than computational cost — Cardinal's statistical methods are the package's core functionality and need different evaluation.
- When validating an exact numerical reproduction is expected — runtime is hardware-dependent, so it cannot be matched to a reference number precisely.
- When the input is already a timing/performance table — the skill measures and records runtimes, it does not re-analyze existing benchmark data.

## Inputs

- Raw (unprocessed) MS imaging spectra, e.g. pig fetus dataset in imzML or Analyze 7.5 format
- Processing configuration (method=PCA, ncomp=20, raw vs. picked peaks)
- Hardware description (CPU model, clock speed, RAM)

## Outputs

- Measured elapsed wall-clock time for the operation (seconds)
- Structured timing record, e.g. pca_timing.csv with columns for method, ncomp, dataset, and elapsed time

## How to apply

Load the target data — e.g. raw (unprocessed) pig fetus MSI spectra in R via Cardinal's readMSIData(), which supports imzML and Analyze 7.5 — then wrap the operation of interest in a timer: request the first 20 principal components with PCA(ncomp=20) around Sys.time() calls. Record the elapsed time into a structured CSV (pca_timing.csv) that names the method (PCA, ncomp=20), the dataset, and the hardware. Validation is magnitude-based, not exact: the recorded time should be comparable to the reported 86.9 s for 20 PCs on the raw pig fetus data (4.3 s on picked peaks), treating absolute runtime as hardware-dependent expert judgment since it was measured on a 2.6 GHz Intel Core i7 MacBook Pro with 16 GB memory. Rationale: Cardinal's backend (the matter package, rewritten for larger-than-memory MS imaging datasets where raw spectra cannot be fully loaded into memory) directly governs performance, so runtime is a meaningful proxy for backend behavior.

## Related tools

- **Cardinal** (R package in which the timed operation (PCA, ncomp=20) is executed on the MSI spectra) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment used to load data, time the call (Sys.time), and write the timing CSV)
- **BiocManager** (Used to install Cardinal via BiocManager::install("Cardinal") before benchmarking)

## Examples

```
t0 <- Sys.time(); pca <- PCA(msi, ncomp=20); dt <- difftime(Sys.time(), t0, units="secs"); write.csv(data.frame(method="PCA",ncomp=20,dataset="pig_fetus_raw",elapsed_s=dt), "pca_timing.csv", row.names=FALSE)
```

## Evaluation signals

- The recorded elapsed time for PCA (ncomp=20) on the raw pig fetus data is of comparable magnitude to the reported 86.9 s (same operation on picked peaks took 4.3 s).
- pca_timing.csv exists and contains the method name, ncomp=20, dataset identifier, and measured elapsed time — not just a bare number.
- The hardware used for the measurement (e.g. 2.6 GHz Intel Core i7, 16 GB RAM) is captured alongside the timing so the number is interpretable.
- Exactly the specified operation (first 20 principal components on raw, unprocessed spectra) was timed — not picked peaks or a different component count.
- Absolute runtime is treated as hardware-dependent expert judgment rather than an exact reproduction target.

## Limitations

- Runtime is hardware-dependent: the reference 86.9 s was measured on a MacBook Pro with a 2.6 GHz Intel Core i7 and 16 GB memory, so comparisons across machines are qualitative only.
- Cardinal 3 dropped legacy support and rewrote the matter backend for larger-than-memory datasets, so timings from older versions are not directly comparable.
- Timing depends on whether spectra are raw (unprocessed) or pre-processed (picked peaks): 86.9 s vs. 4.3 s for the same PCA operation — the data state must be recorded.

## Evidence

- [intro] the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten"
- [intro] raw spectra cannot be fully loaded into memory: "the basic assumption that the raw spectra can be fully loaded into memory is unreasonable for many MS imaging experiments today."
- [readme] traditional tools like PCA and PLS: [evidence span withheld — anchor status: external_doc]
