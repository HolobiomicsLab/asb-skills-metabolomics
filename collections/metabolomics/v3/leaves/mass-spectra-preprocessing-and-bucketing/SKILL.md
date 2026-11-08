---
name: mass-spectra-preprocessing-and-bucketing
description: Use when raw MS/MS MGF files must be cleaned (peak filtering, intensity
  normalization, short/low-peak spectra dropped) and partitioned into precursor-m/z
  buckets prior to pairwise-distance clustering.
when_to_use_negative:
- Input is already preprocessed/normalized or not in MGF peak-file form — HyperSpec's
  parser and preprocessor expect MGF files with precursor, peak, and intensity records.
- The analysis targets peptide identification on raw spectra rather than clustering
  — this skill only merges and organizes spectra, it does not search them against
  a protein database.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Provides the Spectrum Data Parser, parallelized Preprocessor, and bucket-division
    stage that implement this skill end-to-end
  repo: https://github.com/wh-xu/Hyper-Spec
- name: graphics processing units
  role: Not used in this skill itself, but the preprocessed, bucketed spectra are
    the direct input to GPU-accelerated HD encoding and Hamming distance computation
    in HyperSpec
- name: Python
  role: Implementation language for HyperSpec's preprocessing and bucketing modules
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:d723b5db59cbac7520af5726de6a8e8459750c5b73e6a1e1cd75db6be0f8a704
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - Python
  - graphics processing units
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-spectra-preprocessing-and-bucketing

## Summary

Preprocess MGF-format MS/MS spectra and bucket them by precursor m/z before HD encoding, as implemented in HyperSpec. Preprocessing removes low-information peaks and spectra while bucketing partitions the dataset so pairwise Hamming distances are computed within bounded, homogeneous buckets.

## When to use

Use when raw MS/MS MGF files must be cleaned (peak filtering, intensity normalization, short/low-peak spectra dropped) and partitioned into precursor-m/z buckets prior to pairwise-distance clustering.

## When NOT to use

- Input is already preprocessed/normalized or not in MGF peak-file form — HyperSpec's parser and preprocessor expect MGF files with precursor, peak, and intensity records.
- The analysis targets peptide identification on raw spectra rather than clustering — this skill only merges and organizes spectra, it does not search them against a protein database.

## Inputs

- MGF-format MS/MS spectrum peak files (e.g. ThermoRawFileParser-converted PXD000561 raw files)

## Outputs

- Cleaned spectra filtered by charge, peak count, and m/z range with intensities normalized to [0,1]
- Precursor-m/z-sorted buckets grouping spectra for per-bucket HD encoding and distance computation
- Parquet clustering-results table with columns bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative

## How to apply

Run HyperSpec's parallelized Spectrum Data Parser plus Preprocessor, reading each MGF file in parallel with multiprocessing across k CPU cores (default 6, --cpu_core_preprocess). Retain only precursor charges 2 and 3 (--cluster_charges 2 3), remove precursor-related peaks and peaks below 1% of the base peak intensity (--min_intensity 0.01), drop spectra with fewer than 5 valid peaks (--min_peaks 5) or a mass range narrower than 250 Da (--min_mz_range 250), keep at most the 50 highest-intensity peaks (--max_peaks_used 50), and L2-normalize intensities to [0,1] (scaling option off/root/log/rank). Sort spectra by ascending precursor m/z and divide them into buckets using the charge-aware bucket formula keyed on --mz_interval, so that each bucket contains spectra of similar precursor m/z and the later per-bucket Hamming distance computation stays bounded. Rationale: spectrum preprocessing contributes 20–90% of total clustering runtime in state-of-the-art tools, so HyperSpec vectorizes these steps; the m/z-sorted buckets also let clustering merge consensus candidates only among comparable precursors.

## Related tools

- **HyperSpec** (Provides the Spectrum Data Parser, parallelized Preprocessor, and bucket-division stage that implement this skill end-to-end) — https://github.com/wh-xu/Hyper-Spec
- **graphics processing units** (Not used in this skill itself, but the preprocessed, bucketed spectra are the direct input to GPU-accelerated HD encoding and Hamming distance computation in HyperSpec)
- **Python** (Implementation language for HyperSpec's preprocessing and bucketing modules)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_charges 2 3 --min_peaks 5 --min_mz_range 250 --max_peaks_used 50 --min_intensity 0.01
```

## Evaluation signals

- Every output spectrum retains ≥5 valid peaks spanning ≥250 Da and its precursor charge is 2 or 3.
- Peak intensities are L2-normalized to [0,1]; only up to the 50 highest-intensity peaks and no peaks <1% of the base peak remain.
- Buckets are ordered by ascending precursor m/z and each spectrum's bucket label is consistent with the charge-aware bucket formula / --mz_interval setting.
- Exported parquet results contain the expected columns: bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative, with exactly one cluster label line per input MS/MS spectrum.

## Limitations

- Preprocessing consumes 20–90% of total runtime in state-of-the-art spectral clustering tools, so it is the dominant bottleneck even when vectorized.
- Thresholds (min 5 peaks, 250 Da range, 50 max peaks, 1% intensity cutoff, charges 2 and 3) are tuned to the draft human proteome Data set-E; other datasets may need recalibration.
- HyperSpec has been tested mainly on NVIDIA GTX 1080Ti and GTX 3090; clustering the large PXD000561 dataset requires the larger-memory GTX 3090, and a multi-GPU mode is not yet implemented.

## Evidence

- [methods] Preprocessor filters and normalizes spectra: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] Min peak count and mass-range thresholds: "Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed."
- [methods] Max peaks retained and intensity normalization: [evidence span withheld — anchor status: not_found]
- [methods] Bucket division by precursor m/z: "the spectra are sorted and organized by ascending precursor m/z order"
- [methods] Parallelized preprocessing motivation: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [readme] Charge restriction in CLI: "--cluster_charges           The charges to be clustered. (default: 2 3)"
