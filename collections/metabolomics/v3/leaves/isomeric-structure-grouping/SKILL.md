---
name: isomeric-structure-grouping
description: Use when benchmarking MS/MS library-search FDR on natural-product or
  isomer-rich libraries, where decoy-like pairs differing by one bond must be isolated
  to measure FDR curves for entropy-similarity thresholds.
when_to_use_negative:
- When the analysis goal is general library search speed or index building (use Flash/Dynamic
  Entropy Search instead) rather than FDR evaluation on similar-structure groups
- When spectra have not been cleaned — raw peaks must first pass clean_spectrum preprocessing
  or group-level similarity scores will be unreliable
- When the library lacks compound structure annotations, since one-bond-difference
  collectives cannot be identified from spectra alone
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ms_entropy
  role: Provides clean_spectrum (noise_threshold=0.01, min_ms2_difference_in_da=0.05)
    and calculate_entropy_similarity at 0.05 Da MS2 tolerance for scoring spectra
    within structure groups
  repo: https://github.com/YuanyueLi/MSEntropy
- name: MSEntropy
  role: Python package supplying the spectral entropy and entropy similarity functions
    underlying the similarity scores evaluated per structure group
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds peak arrays as np.ndarray([[m/z, intensity], ...], dtype=np.float32)
    and supports the one-bond-difference similarity implementation
- name: SciPy
  role: Supports the one-bond-difference entropy-similarity variant implementation
    in Python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:dd42e7b4b92a558c47b56a72fde866d9d1d42e8fdd94bae02564c5141b6e0dfe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python
  - SciPy
  - ms_entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# isomeric-structure-grouping

## Summary

Groups library MS/MS spectra into collectives of highly similar structures whose compounds differ by a single bond, so that false discovery rates of spectral matching can be evaluated separately for isomer/isobar-confusable pairs.

## When to use

Use when benchmarking MS/MS library-search FDR on natural-product or isomer-rich libraries, where decoy-like pairs differing by one bond must be isolated to measure FDR curves for entropy-similarity thresholds.

## When NOT to use

- When the analysis goal is general library search speed or index building (use Flash/Dynamic Entropy Search instead) rather than FDR evaluation on similar-structure groups
- When spectra have not been cleaned — raw peaks must first pass clean_spectrum preprocessing or group-level similarity scores will be unreliable
- When the library lacks compound structure annotations, since one-bond-difference collectives cannot be identified from spectra alone

## Inputs

- Experimental MS/MS spectra from MassBank of North America (e.g. 37,299 natural-product spectra) as [m/z, intensity] peak arrays with precursor m/z
- Library spectra from NIST20 and/or GNPS with associated compound structures
- Structure records (e.g. SMILES/InChI) for library compounds to identify one-bond-difference pairs

## Outputs

- Grouped collectives of highly similar structures (one-bond-difference pairs/multiplets)
- FDR-versus-threshold curves for baseline entropy similarity and the one-bond-difference variant
- FDR table written to one_bond_fdr_results.csv

## How to apply

Load experimental MS/MS spectra from MassBank of North America and, where available, NIST20 into [m/z, intensity] peak arrays, then clean each with ms_entropy.clean_spectrum (remove empty peaks, remove ions above precursor m/z - 1.6 Da, centroid within +/- 0.05 Da, drop peaks below 1% of base-peak intensity, normalize intensities to sum 1). Group library spectra into collectives of highly similar structures, identifying compound pairs or multiplets whose structures differ by a single bond — these groups act as the hardest negative (decoy-like) test cases. Compute baseline entropy similarities within each group using ms_entropy.calculate_entropy_similarity at 0.05 Da MS2 tolerance, optionally alongside a one-bond-difference weighted variant implemented in NumPy/SciPy. Sweep similarity thresholds within the groups and record false discovery rates at each threshold, yielding an FDR-versus-threshold table (e.g. one_bond_fdr_results.csv) that can be compared with the reported criterion of FDR < 10% at entropy similarity score 0.75.

## Related tools

- **ms_entropy** (Provides clean_spectrum (noise_threshold=0.01, min_ms2_difference_in_da=0.05) and calculate_entropy_similarity at 0.05 Da MS2 tolerance for scoring spectra within structure groups) — https://github.com/YuanyueLi/MSEntropy
- **MSEntropy** (Python package supplying the spectral entropy and entropy similarity functions underlying the similarity scores evaluated per structure group) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds peak arrays as np.ndarray([[m/z, intensity], ...], dtype=np.float32) and supports the one-bond-difference similarity implementation)
- **SciPy** (Supports the one-bond-difference entropy-similarity variant implementation in Python)

## Examples

```
import ms_entropy as me; clean_peaks = me.clean_spectrum(peaks, min_ms2_difference_in_da=0.05, noise_threshold=0.01); sim = me.calculate_entropy_similarity(clean_peaks_query, clean_peaks_library, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- All computed FDR values must lie between 0 and 1
- The baseline entropy-similarity curve should reproduce FDR < 10% at an entropy similarity score of 0.75, matching the reported acceptance criterion
- Cleaned spectra must have intensities normalized to sum 1 and no peaks above precursor m/z - 1.6 Da
- Each structure collective should contain only pairs/multiplets verified to differ by a single bond; dot-product baseline can be computed alongside as a comparison reference

## Limitations

- Even with entropy similarity scores, no single similarity threshold can be defined that would prevent all false-positive discoveries
- The 0.75 threshold with FDR < 10% was established on natural-product experimental spectra; it may not transfer to other compound classes
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values

## Evidence
