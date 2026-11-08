---
name: proteomics-dataset-subset-sampling
description: Use when evaluating the computational scalability (runtime, memory) of
  a spectrum clustering or search tool across progressively larger subsets of a mass
  spectrometry dataset.
when_to_use_negative:
- The goal is clustering quality (purity/completeness) comparison rather than runtime/memory
  scaling — use a clustering-accuracy benchmark instead
- The dataset is too small to exhibit meaningful scaling behavior (a single run with
  no subset series cannot demonstrate linear vs quadratic scaling)
- Input data is metabolomics or top-down spectra, where falcon's default bottom-up
  proteomics preprocessing settings (min_peaks 5, min_mz_range 250, m/z 101–500) must
  be adjusted first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose runtime and memory consumption are measured
    across dataset subsets
  repo: https://github.com/bittremieux/falcon
- name: Python 3.8
  role: Runtime environment for falcon and for tabulating/plotting the scaling results
    (CSV, matplotlib)
- name: ThermoRawFileParser
  role: Converted the raw files from PRIDE PXD000561 to MGF peak files prior to subsetting
    and clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:405d78cd2002e2ddc5f60939e6a5d2770354f470dd99f727aa96744c04c9a88b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python 3.8
  - ThermoRawFileParser
  - falcon
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# proteomics-dataset-subset-sampling

## Summary

Randomly sample subsets of peak files from a large proteomics dataset (e.g. 10 to 2212 MGF files from the Kim et al. human draft proteome) to benchmark how a tool's runtime and memory scale with input size.

## When to use

Use when evaluating the computational scalability (runtime, memory) of a spectrum clustering or search tool across progressively larger subsets of a mass spectrometry dataset.

## When NOT to use

- The goal is clustering quality (purity/completeness) comparison rather than runtime/memory scaling — use a clustering-accuracy benchmark instead
- The dataset is too small to exhibit meaningful scaling behavior (a single run with no subset series cannot demonstrate linear vs quadratic scaling)
- Input data is metabolomics or top-down spectra, where falcon's default bottom-up proteomics preprocessing settings (min_peaks 5, min_mz_range 250, m/z 101–500) must be adjusted first

## Inputs

- Converted MGF peak files from the Kim et al. human draft proteome dataset (PRIDE PXD000561), 2212 files / 25 million spectra total
- Random subset definitions of peak files (10, 50, 100, 500, 1000, 1500, 2000, 2212 files)
- Tool configuration parameters (precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.10 for falcon)

## Outputs

- scaling_runtime.csv with columns: subset size (files), number of spectra, runtime, peak memory
- Matplotlib plot of runtime versus number of input spectra showing the scaling curve
- Per-subset falcon clustering outputs (cluster assignments in falcon.csv, optional representative spectra in falcon.mgf)

## How to apply

Split the converted MGF (or mzML) peak files of the dataset — here the Kim et al. human draft proteome dataset (PRIDE PXD000561, 25 million spectra) — into random subsets spanning small to full size, e.g. 10, 50, 100, 500, 1000, 1500, 2000, and 2212 files. Run the target tool on each subset with identical, paper-derived settings (for falcon: precursor tolerance 20 ppm, fragment tolerance 0.05 Da, cosine distance threshold eps 0.10) and record total runtime and peak memory per run. Tabulate subset size, number of spectra, runtime, and memory in a CSV (e.g. scaling_runtime.csv) and plot runtime versus number of input spectra with matplotlib. The rationale is that although pairwise spectrum comparisons scale quadratically with the number of spectra, the tool under test (falcon) should exhibit optimal linear scaling thanks to its feature hashing, nearest neighbor indexing, and sparse pairwise distance matrix construction. Validate that the observed runtime-versus-spectra curve reproduces the reported near-linear trend and that the largest run completes in hours on a multi-core node (e.g. two 14-core Intel E5-2680v4 CPUs, 128 GB memory).

## Related tools

- **falcon** (Spectrum clustering tool whose runtime and memory consumption are measured across dataset subsets) — https://github.com/bittremieux/falcon
- **Python 3.8** (Runtime environment for falcon and for tabulating/plotting the scaling results (CSV, matplotlib))
- **ThermoRawFileParser** (Converted the raw files from PRIDE PXD000561 to MGF peak files prior to subsetting and clustering)

## Examples

```
falcon subset_2212/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- The runtime-versus-number-of-spectra plot shows a near-linear trend across subsets of 10 to 2212 files, matching the paper's reported linear scaling
- Each subset's runtime and memory are recorded for every subset size with no missing values in scaling_runtime.csv
- The 2212-file run (25 million spectra) completes within the order of hours on a multi-core compute node (e.g. two 14-core Intel E5-2680v4 CPUs, 128 GB memory)
- All subsets are clustered with identical parameters (precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.10) so that only dataset size varies
- Subsets are randomly sampled, so re-drawing subsets yields comparable scaling behavior (no pathological file selection)

## Limitations

- Although pairwise spectrum comparisons scale quadratically with the number of processed spectra, the observed linear runtime scaling holds for the tested range of 10 to 2212 peak files and may not extend beyond this dataset
- falcon requires Python 3.8+ and is available only on Linux and OSX platforms
- The eps cosine distance threshold depends on spectral characteristics and preprocessing; the scaling benchmark fixes eps 0.10, but other values (0.05–0.15 typical, up to 0.30 for aggressive clustering) may alter results

## Evidence

- [methods] Raw files were downloaded from PRIDE (project PXD000561): [evidence span withheld — anchor status: not_found]
