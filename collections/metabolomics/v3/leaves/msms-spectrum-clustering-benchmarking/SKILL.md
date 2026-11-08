---
name: msms-spectrum-clustering-benchmarking
description: Use when quantifying the efficiency of a spectrum clustering tool on
  large MS/MS datasets — e.g. comparing nearest-neighbor search cost (~47 comparisons/spectrum
  for falcon) to exhaustive pairwise comparison within a 20 ppm precursor tolerance,
  or timing clustering of millions of spectra.
when_to_use_negative:
- When the goal is comparing clustering quality (purity/completeness) rather than
  computational cost — use the article's quality comparison workflow instead
- When clustering metabolomics or top-down data with falcon defaults — the README
  notes default preprocessing settings are intended for bottom-up proteomics and must
  be adjusted (e.g. lower min_peaks and min_mz_range)
- When the input is not raw peak list data (mzML, mzXML, or MGF) but already-clustered
  or consensus spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose nearest-neighbor searching is instrumented
    to count comparisons per spectrum and timed end-to-end
  repo: https://github.com/bittremieux/falcon
- name: msCRUSH
  role: Alternative hashing-based clustering tool run on the same data to provide
    a runtime comparison point
- name: ThermoRawFileParser
  role: Converts raw Thermo files to MGF format as falcon/msCRUSH input (version 1.2.3)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:e6031986b6837d7d77c58d008e87e77c1c54697055484db8422d3efda72b3e09
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python
  - ThermoRawFileParser
  - falcon
  - msCRUSH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msms-spectrum-clustering-benchmarking

## Summary

Benchmarks the computational cost of falcon's fast nearest-neighbor-based MS/MS spectrum clustering (wall-clock runtime and average spectrum–spectrum comparisons per spectrum) against brute-force pairwise comparison and the msCRUSH hashing-based approach on a large proteomics dataset.

## When to use

Use when quantifying the efficiency of a spectrum clustering tool on large MS/MS datasets — e.g. comparing nearest-neighbor search cost (~47 comparisons/spectrum for falcon) to exhaustive pairwise comparison within a 20 ppm precursor tolerance, or timing clustering of millions of spectra.

## When NOT to use

- When the goal is comparing clustering quality (purity/completeness) rather than computational cost — use the article's quality comparison workflow instead
- When clustering metabolomics or top-down data with falcon defaults — the README notes default preprocessing settings are intended for bottom-up proteomics and must be adjusted (e.g. lower min_peaks and min_mz_range)
- When the input is not raw peak list data (mzML, mzXML, or MGF) but already-clustered or consensus spectra

## Inputs

- Raw Thermo mass spectrometry files from PRIDE PXD000561 (2212 files, human draft proteome dataset)
- MGF-format peak files converted from the raw data via ThermoRawFileParser 1.2.3
- falcon configuration: precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.05
- msCRUSH configuration: 100 clustering iterations, 15 hash functions per hash table

## Outputs

- runtime_benchmark.csv containing falcon total runtime (~5 hours), average spectrum–spectrum comparisons per spectrum (~47), and msCRUSH total runtime (~4 hours)
- falcon cluster assignments (comma-separated file with each MS/MS spectrum and its cluster label)
- Instrumentation log of spectrum–spectrum comparisons executed during nearest-neighbor searching

## How to apply

Download the human draft proteome raw files of Kim et al. (PRIDE PXD000561, 2212 files, ~25 million spectra) and convert them to MGF with ThermoRawFileParser 1.2.3. Run falcon on the full dataset with precursor_tol 20 ppm, fragment_tol 0.05 Da, and eps 0.05 (the cosine distance threshold yielding approximately 1% incorrectly clustered spectra), instrumented to count spectrum–spectrum comparisons during nearest-neighbor searching; falcon performs feature hashing to low-dimensional vectors, Voronoi-diagram inverted-index searching, sparse pairwise distance matrix construction, and DBSCAN clustering, averaging only ~47 comparisons per spectrum and ~5 hours total runtime. As a context point, run msCRUSH (100 clustering iterations, 15 hash functions per hash table) on the same data, all cores, recording its ~4 hour runtime. Record both tools' runtimes and falcon's average comparisons per spectrum into runtime_benchmark.csv, using all cores of a two 14-core Intel E5-2680v4 / 128 GB memory node.

## Related tools

- **falcon** (Spectrum clustering tool whose nearest-neighbor searching is instrumented to count comparisons per spectrum and timed end-to-end) — https://github.com/bittremieux/falcon
- **msCRUSH** (Alternative hashing-based clustering tool run on the same data to provide a runtime comparison point)
- **ThermoRawFileParser** (Converts raw Thermo files to MGF format as falcon/msCRUSH input (version 1.2.3))

## Examples

```
falcon peak/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- falcon total runtime is on the order of ~5 hours on a node with two 14-core Intel E5-2680v4 CPUs and 128 GB memory using all cores
- Average spectrum–spectrum comparisons during nearest-neighbor searching is approximately 47 per spectrum (instead of exhaustively comparing all spectra within the 20 ppm precursor tolerance)
- msCRUSH runtime on the identical data is approximately 4 hours
- falcon configuration with eps 0.05 yields approximately 1% incorrectly clustered spectra
- All 2212 peak files of PXD000561 (~25 million spectra) were processed; runtime_benchmark.csv contains falcon and msCRUSH entries

## Limitations

- Benchmark figures (~5 h falcon, ~4 h msCRUSH, ~47 comparisons/spectrum) are specific to the Kim et al. human draft proteome dataset (25 million spectra) and the 28-core E5-2680v4 hardware; other datasets or machines will differ
- falcon requires Python 3.8+ and runs on Linux and OSX only
- The eps cosine distance threshold crucially governs cluster purity and its ideal value depends on the spectral characteristics of the data and the preprocessing/scaling configuration

## Evidence

- [methods] Raw files from PRIDE PXD000561 converted to MGF: [evidence span withheld — anchor status: not_found]
