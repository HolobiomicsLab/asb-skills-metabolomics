---
name: peak-intensity-l2-normalization
description: Use when MS/MS peak lists (e.g., parsed from MGF) must be intensity-normalized
  before spectral similarity comparison or clustering, and raw base-peak intensity
  scales vary across spectra.
when_to_use_negative:
- When a nonlinear intensity transform (root, log, or rank scaling, also exposed via
  HyperSpec's --scaling option) is specifically required for the downstream comparison
  instead of linear L2 normalization.
- When spectra have not yet passed the filtering chain (precursor-peak removal, min_peaks=5,
  min_mz_range=250, max_peaks_used=50); normalizing unfiltered intensities reintroduces
  noise and precursor peaks into the vector.
- When intensities must retain absolute abundance information, e.g., for quantitative
  label-free analysis rather than spectral similarity clustering.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Provides the parallelized Numba preprocessor that applies L2 normalization
    (--scaling) to filtered peak intensities within its spectrum preprocessing module.
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: Runtime environment in which the vectorized normalization kernels (Numba JIT-compiled,
    GPU/CPU) are implemented.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:34435b75b4d693ece3873618c3eaa22499c9244915fd734c6a8e05777eb93007
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - Python
  - ThermoRawFileParser
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Peak intensity L2 normalization

## Summary

Normalizes filtered MS/MS peak intensities to the [0, 1] range using the L2 norm as the final step of HyperSpec's spectrum preprocessing, so downstream HD encoding and Hamming-distance-based clustering operate on scale-invariant peak vectors.

## When to use

Use when MS/MS peak lists (e.g., parsed from MGF) must be intensity-normalized before spectral similarity comparison or clustering, and raw base-peak intensity scales vary across spectra.

## When NOT to use

- When a nonlinear intensity transform (root, log, or rank scaling, also exposed via HyperSpec's --scaling option) is specifically required for the downstream comparison instead of linear L2 normalization.
- When spectra have not yet passed the filtering chain (precursor-peak removal, min_peaks=5, min_mz_range=250, max_peaks_used=50); normalizing unfiltered intensities reintroduces noise and precursor peaks into the vector.
- When intensities must retain absolute abundance information, e.g., for quantitative label-free analysis rather than spectral similarity clustering.

## Inputs

- Filtered peak m/z and intensity arrays per MS/MS spectrum (post precursor-peak removal, min_intensity filtering, min_peaks/min_mz_range filtering, and top-50 peak retention)
- MGF-derived spectrum metadata (identifier, scan, precursor charge, precursor m/z, retention time)

## Outputs

- Per-spectrum intensity vectors with values in [0, 1] (L2-normalized)
- Preprocessed spectra written to intermediate files for HD encoding and m/z-sorted bucket division

## How to apply

After the preceding HyperSpec filter steps (remove precursor-related peaks and peaks below min_intensity=1% of the base peak, drop spectra with <5 valid peaks or <250 Da mass range, retain at most the 50 highest-intensity peaks via max_peaks_used=50), compute the L2 norm of each spectrum's intensity vector and divide every peak intensity by it. This maps intensities into [0, 1], making each spectrum's peak vector scale-invariant regardless of absolute ion abundance. In the HyperSpec CLI this normalization is controlled by the --scaling option ({off,root,log,rank}); the L2-norm scaling described in the paper is applied as configured and is executed inside the Numba JIT-compiled parallel preprocessor, not in plain Python. Write the normalized m/z and intensity arrays, together with spectrum metadata (identifier, scan, precursor charge, precursor m/z, retention time), to intermediate files for the HD encoding stage.

## Related tools

- **HyperSpec** (Provides the parallelized Numba preprocessor that applies L2 normalization (--scaling) to filtered peak intensities within its spectrum preprocessing module.) — https://github.com/wh-xu/Hyper-Spec
- **Python** (Runtime environment in which the vectorized normalization kernels (Numba JIT-compiled, GPU/CPU) are implemented.)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=6 --max_peaks_used=50 --min_intensity=0.01 --scaling=rank --cluster_charges 2 3
```

## Evaluation signals

- Every normalized intensity value lies within [0, 1] inclusive.
- For each spectrum, the L2 norm of the output intensity vector equals 1 (within floating-point tolerance).
- No spectrum with fewer than 5 valid peaks or a <250 Da mass range survives; at most 50 peaks are retained per spectrum before normalization.
- Peak count and m/z arrays are unchanged by normalization (only intensities are rescaled), and metadata fields (identifier, scan, precursor_charge, precursor_mz, retention_time) round-trip to intermediate files.
- Preprocessing throughput scales with the number of CPU cores (cpu_core_preprocess, default 6) and the preprocessing speed advantage grows on larger data sets as reported in Figure 8a.

## Limitations

- L2 normalization is linear, so relative peak-intensity ratios within a spectrum are preserved but cross-spectrum absolute abundance information is discarded.
- Spectrum preprocessing (including this normalization) contributes 20-90% of overall runtime in state-of-the-art clustering tools, so serial implementations become the bottleneck on large data sets; the vectorized/parallel implementation is essential.
- Scaling behavior depends on the --scaling configuration; if set to 'off', intensities are not normalized to [0, 1].

## Evidence

- [methods] L2-normalize peak intensities: [evidence span withheld — anchor status: not_found]
- [readme] scaling option: "--scaling {off,root,log,rank}"
- [methods] vectorized preprocessor: [evidence span withheld — anchor status: not_found]
- [abstract] preprocessing bottleneck: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
