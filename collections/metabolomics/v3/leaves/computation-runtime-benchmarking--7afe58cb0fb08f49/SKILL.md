---
name: computation-runtime-benchmarking
description: Use when comparing computational cost of an MSI analysis between raw
  and peak-picked spectra, or when documenting runtime of a Cardinal computation as
  evidence that pre-processing yields a speedup.
when_to_use_negative:
- The analysis goal is statistical interpretation of the PCA/segmentation results
  rather than measuring computation cost — use the analysis skill, not the benchmarking
  skill.
- The dataset is small enough to load fully into memory and timing is not a stated
  objective — runtime benchmarking adds no evidentiary value.
- Input spectra have not been peak-picked and the question is about pre-processing
  quality, not runtime comparison.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cardinal
  role: Provides PCA() for computing principal components on MSI spectra and the peak-picking
    pre-processing that generates the fast (peak-picked) input state being benchmarked
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Execution environment; system.time() is the timing mechanism wrapping the
    PCA call
- name: matter
  role: Cardinal's backend enabling computing on larger-than-memory MS imaging datasets,
    which affects the absolute runtimes being measured
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:04ee1245c6a0fa319099fdde5d3fb1cbaec349efb5a4413d230d46a95b54e8fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - R
  - matter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# computation-runtime-benchmarking

## Summary

Measures the elapsed wall-clock time of a Cardinal statistical computation (e.g. PCA of the first 20 principal components) on mass spectrometry imaging spectra, and compares timing between peak-picked and raw data to quantify the speedup from pre-processing.

## When to use

Use when comparing computational cost of an MSI analysis between raw and peak-picked spectra, or when documenting runtime of a Cardinal computation as evidence that pre-processing yields a speedup.

## When NOT to use

- The analysis goal is statistical interpretation of the PCA/segmentation results rather than measuring computation cost — use the analysis skill, not the benchmarking skill.
- The dataset is small enough to load fully into memory and timing is not a stated objective — runtime benchmarking adds no evidentiary value.
- Input spectra have not been peak-picked and the question is about pre-processing quality, not runtime comparison.

## Inputs

- Peak-picked (processed) pig fetus MSI spectra from Cardinal's peakPick/peakProcess step
- Raw (unprocessed) MSI spectra for the same dataset, for the baseline timing comparison

## Outputs

- Elapsed-time measurement from system.time() for each data state and component count
- pca_timing.csv — structured timing record containing data state (peak-picked/raw) and component count (nComp = 20)
- Speedup ratio of peak-picked vs raw PCA runtime (~4.3 s vs ~86.9 s in the reference run)

## How to apply

Load the peak-picked (processed) MSI spectra produced by Cardinal's peak-picking step (peakPick/peakProcess), then call PCA() requesting the first 20 principal components (nComp = 20), wrapping the call in R's system.time() to capture elapsed computation. Repeat the identical call on the raw spectra under the same hardware so the comparison is controlled (reference: 4.3 s peak-picked vs 86.9 s raw on a MacBook Pro with a 2.6 GHz Intel Core i7 and 16 GB memory). Write the elapsed time into a structured record file, pca_timing.csv, annotating the data state (peak-picked vs raw) and the component count so the benchmark is reproducible and auditable. Rationale: peak picking reduces the spectral data volume before statistical analysis, and Cardinal's matter backend is designed for computing on larger-than-memory MS imaging datasets, so runtime is the metric that demonstrates the benefit.

## Related tools

- **Cardinal** (Provides PCA() for computing principal components on MSI spectra and the peak-picking pre-processing that generates the fast (peak-picked) input state being benchmarked) — github.com/kuwisdelu/Cardinal
- **R** (Execution environment; system.time() is the timing mechanism wrapping the PCA call)
- **matter** (Cardinal's backend enabling computing on larger-than-memory MS imaging datasets, which affects the absolute runtimes being measured)

## Examples

```
system.time(pca <- PCA(processed_spectra, nComp=20))
```

## Evaluation signals

- pca_timing.csv exists and contains one row per data state with an elapsed-time field, data-state label (peak-picked/raw), and component count (20).
- Peak-picked PCA elapsed time is on the order of seconds (~4.3 s), substantially lower than the raw-spectra time (~86.9 s), yielding a large speedup ratio.
- The same PCA call (nComp = 20) and hardware are used for both data states so the timing comparison is controlled.
- The benchmarked computation runs to completion without errors on the peak-picked spectra, confirming the upstream peak-picking step produced valid input.

## Limitations

- Runtimes are hardware-dependent — the reference numbers (4.3 s vs 86.9 s) were measured on a MacBook Pro with a 2.6 GHz Intel Core i7 and 16 GB memory and will not transfer directly to other machines.
- Absolute timings are version-dependent: Cardinal 3.6 is a major update with breaking changes and a rewritten matter backend, so legacy timings may not reproduce on newer versions.
- Only the first 20 principal components were benchmarked; timing does not necessarily scale linearly with component count or dataset size.

## Evidence

- [results] Reference runtime comparison: [evidence span withheld — anchor status: not_found]
- [readme] Cardinal's core scope: [evidence span withheld — anchor status: external_doc]
- [intro] Peak picking step: "`peakPick()` for peak detection and summarization"
- [abstract] Cardinal's purpose: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [intro] matter backend: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten"
