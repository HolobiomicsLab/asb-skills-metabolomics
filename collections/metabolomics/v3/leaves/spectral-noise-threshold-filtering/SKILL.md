---
name: spectral-noise-threshold-filtering
description: Use when raw MS/MS query or library spectra (.mgf, .msp, .mzML peak lists)
  contain noise peaks that would corrupt spectral matching — i.e., before entropy-similarity
  search or when flagging poor-quality spectra.
when_to_use_negative:
- Spectra already centroided and normalized by a prior clean_spectrum pass — double
  cleaning is unnecessary and may distort intensities
- When using DynamicEntropySearch.add_new_spectra with its built-in cleaning (clean=True
  default); external cleaning then applies only if clean=False
- When performing raw unweighted dot-product comparisons on unprocessed vendor profiles
  — this skill's parameters are tuned for entropy similarity
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms_entropy
  role: Provides clean_spectrum() and read_one_spectrum() used to implement the noise-threshold
    filtering and normalization steps
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds spectra as 2D float32 peak arrays passed to clean_spectrum
- name: DynamicEntropySearch
  role: Alternative entry point whose add_new_spectra() includes a built-in cleaning
    function equivalent to clean_spectrum
  repo: https://github.com/2bereal-me/DynamicEntropySearch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:9405729c6c4b097922415108dd45a1b407469c437d7450c49fc4b476624977f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - DynamicEntropySearch
  - NumPy
  - Python
  - SciPy
  - ms_entropy
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# spectral-noise-threshold-filtering

## Summary

Pre-processing of MS/MS peak lists to remove low-abundance noise ions before entropy-similarity library search, implemented via ms_entropy.clean_spectrum. Cleaning (noise removal, precursor-ion removal, centroiding, normalization) is required for the robust entropy similarity scoring that outperformed dot-product similarity in the NIST20 benchmark.

## When to use

Use when raw MS/MS query or library spectra (.mgf, .msp, .mzML peak lists) contain noise peaks that would corrupt spectral matching — i.e., before entropy-similarity search or when flagging poor-quality spectra.

## When NOT to use

- Spectra already centroided and normalized by a prior clean_spectrum pass — double cleaning is unnecessary and may distort intensities
- When using DynamicEntropySearch.add_new_spectra with its built-in cleaning (clean=True default); external cleaning then applies only if clean=False
- When performing raw unweighted dot-product comparisons on unprocessed vendor profiles — this skill's parameters are tuned for entropy similarity

## Inputs

- MS/MS spectra as 2D peak arrays [[m/z, intensity], ...] (np.float32) or files in .mgf, .msp, or .mzML format
- Precursor m/z per spectrum (float) for precursor-ion removal

## Outputs

- Cleaned, centroided, intensity-normalized peak lists (intensities summing to 1) ready for FlashEntropySearch/DynamicEntropySearch indexing or search
- Implicit quality signal: queries dominated by noise (many peaks <1% base peak) can be flagged as poor-quality spectra downstream

## How to apply

Load spectra with ms_entropy.read_one_spectrum, then apply ms_entropy.clean_spectrum per spectrum. The function removes empty peaks (m/z <= 0 or intensity <= 0), removes precursor-region ions with m/z above precursor_mz - precursor_ions_removal_da (1.6 Da, validated on NIST20 and Massbank.us), centroids peaks by merging within +/- min_ms2_difference_in_da (0.05 Da, kept > 2*max MS2 tolerance), removes peaks below noise_threshold * maximum intensity (noise_threshold=0.01, i.e. <1% of base peak), optionally retains only max_peak_num peaks, and normalizes intensities to sum to 1. Apply identical cleaning to both query and reference library spectra so peak matching at ms2_tolerance_in_da=0.02 is symmetric. Note that DynamicEntropySearch.add_new_spectra has a built-in cleaning step, so skip external cleaning only when clean=False is passed there.

## Related tools

- **ms_entropy** (Provides clean_spectrum() and read_one_spectrum() used to implement the noise-threshold filtering and normalization steps) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds spectra as 2D float32 peak arrays passed to clean_spectrum)
- **DynamicEntropySearch** (Alternative entry point whose add_new_spectra() includes a built-in cleaning function equivalent to clean_spectrum) — https://github.com/2bereal-me/DynamicEntropySearch

## Examples

```
import ms_entropy as me; cleaned = me.clean_spectrum(peaks, precursor_mz=precursor, precursor_ions_removal_da=1.6, noise_threshold=0.01, min_ms2_difference_in_da=0.05)
```

## Evaluation signals

- All remaining peaks have intensity >= 1% of the base peak (noise_threshold=0.01) and no peaks exceed precursor_mz - 1.6 Da
- Cleaned spectrum intensities sum to 1 (normalization invariant)
- No duplicate/merged peaks remain within 0.05 Da after centroiding (min_ms2_difference_in_da=0.05)
- Cleaning is applied identically to query and library spectra before matching at ms2_tolerance_in_da=0.02; mismatched cleaning leads to errors in DynamicEntropySearch
- Downstream entropy-similarity matching at score threshold 0.75 achieves FDR below 10% on experimental gut metabolome data

## Limitations

- The <1% base-peak cutoff and 0.05 Da merging tolerance are fixed defaults validated on NIST20/Massbank.us; other thresholds must be re-validated
- Precursor-ion removal at precursor_mz - 1.6 Da is empirically justified for the tested databases, not a universal rule
- Skipping cleaning when constructing a DynamicEntropySearch index (clean=False without external clean_spectrum) can lead to errors

## Evidence
