---
name: mass-spectra-peak-filtering
description: Use when raw MGF MS/MS spectra must be cleaned before clustering or spectral-library
  building and peak information dominates runtime (>95% of raw-file data volume).
when_to_use_negative:
- Input spectra are already preprocessed/clustered — re-filtering would apply the
  1% base-peak and 5-peak/250-Da cutoffs redundantly
- Spectra lack precursor ion metadata, since the precursor-related peak removal step
  depends on precursor information
- The downstream workflow requires spectra with very low signal content preserved
  (the <5 valid peaks and <250 Da range filters intentionally discard them)
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3365
tools:
- name: HyperSpec
  role: Provides the parallelized Numba preprocessor and CLI filter flags (remove_precursor_tol,
    min_intensity, min_peaks, min_mz_range, max_peaks_used, scaling) used to apply
    this filtering chain
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Numba
  role: JIT compilation backend that vectorizes the filtering computation across CPU
    cores
- name: ThermoRawFileParser
  role: Listed as a task tool for raw MS file handling upstream of MGF-based filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:377f09f44dd2eb72a47d3fc27c55d1474736c61d996a25575159521c79ff3414
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - Numba
  - Python
  - ThermoRawFileParser
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectra-peak-filtering

## Summary

Fine-grained peak- and spectrum-level filtering of MS/MS spectra during preprocessing for spectral clustering, as implemented in HyperSpec's parallelized preprocessor: precursor/low-intensity peak removal, minimum-peak-count and mass-range spectrum filters, top-50 peak retention, and L2 intensity normalization.

## When to use

Use when raw MGF MS/MS spectra must be cleaned before clustering or spectral-library building and peak information dominates runtime (>95% of raw-file data volume).

## When NOT to use

- Input spectra are already preprocessed/clustered — re-filtering would apply the 1% base-peak and 5-peak/250-Da cutoffs redundantly
- Spectra lack precursor ion metadata, since the precursor-related peak removal step depends on precursor information
- The downstream workflow requires spectra with very low signal content preserved (the <5 valid peaks and <250 Da range filters intentionally discard them)

## Inputs

- Raw MGF peak files in an input directory (one or many files, read in parallel)
- CLI filter parameters: remove_precursor_tol, min_intensity (1%), min_peaks (5), min_mz_range (250 Da), max_peaks_used (50), scaling mode

## Outputs

- Preprocessed spectra with per-spectrum identifier, scan, precursor charge, precursor m/z, retention time, and filtered/normalized peak m/z and intensity arrays
- Intermediate files ready for ascending precursor m/z sorting, bucket division, and HD encoding

## How to apply

Parse each MGF file in parallel across k independent CPU cores (default 6 via --cpu_core_preprocess) using the Cython-exposed Boost Spirit X3 C++ parser, then apply the filter chain in order on the Numba JIT-compiled preprocessor: (1) remove peaks related to the precursor ion (--remove_precursor_tol) and peaks below 1% of the base peak intensity (--min_intensity); (2) discard spectra with fewer than 5 valid peaks (--min_peaks=5) or a mass range below 250 Da between minimum and maximum peaks (--min_mz_range=250); (3) retain at most the 50 highest-intensity peaks (--max_peaks_used=50) and normalize intensities to [0, 1] by their L2 norm (--scaling, configurable to off/root/log/rank). The rationale is that spectrum preprocessing contributes 20–90% of overall clustering runtime, so vectorized parallel filtering is the key bottleneck optimization. Write preprocessed spectra (identifier, scan, precursor charge, precursor m/z, retention time, filtered peak m/z and intensity arrays) to intermediate files before bucket division by ascending precursor m/z and HD encoding.

## Related tools

- **HyperSpec** (Provides the parallelized Numba preprocessor and CLI filter flags (remove_precursor_tol, min_intensity, min_peaks, min_mz_range, max_peaks_used, scaling) used to apply this filtering chain) — https://github.com/wh-xu/Hyper-Spec
- **Numba** (JIT compilation backend that vectorizes the filtering computation across CPU cores)
- **ThermoRawFileParser** (Listed as a task tool for raw MS file handling upstream of MGF-based filtering)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --min_peaks=5 --min_mz_range=250 --remove_precursor_tol=0.005 --min_intensity=0.01 --max_peaks_used=50 --scaling=off
```

## Evaluation signals

- No spectrum surviving the filter has fewer than 5 valid peaks or a mass range <250 Da between its minimum and maximum peaks
- Each surviving spectrum retains at most 50 peaks, which are the highest-intensity ones
- After L2 normalization all peak intensities fall within [0, 1]
- Peaks related to the precursor ion or below 1% of base peak intensity are absent from the output peak arrays
- Preprocessing runtime scales with k CPU cores (--cpu_core_preprocess, default 6) and grows more favorable relative to alternatives on larger data sets (Figure 8a behavior)

## Limitations

- Preprocessing is a critical bottleneck contributing 20–90% of overall runtime in state-of-the-art spectral clustering tools, so filtering must remain vectorized to be worthwhile
- Peak information occupies over 95% of raw-file data volume, making parsing itself the dominant cost; filtering alone does not remove the parser bottleneck
- Defaults were tuned for the paper's MGF datasets (e.g., PXD000561); the CLI exposes scaling modes (off/root/log/rank) and min_peaks/min_mz_range that change filtering behavior if altered

## Evidence

- [methods] precursor/low-intensity peak removal: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] spectrum-level validity filters: "Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed."
- [methods] top-peak retention and normalization: [evidence span withheld — anchor status: not_found]
- [methods] parallel multiprocessing rationale: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [readme] MGF input and CLI usage: "_HyperSpec_ supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file"
