---
name: msms-spectra-merging-and-renormalization
description: Use when two centroided MS/MS peak lists (m/z, intensity) must be combined
  into a merged spectrum AB to compute spectral entropy similarity, e.g. for library
  matching or compound identification scoring.
when_to_use_negative:
- Spectra have not been centroided and entropy-normalized first (intensities do not
  sum to 1) — merge before normalization invalidates the entropy calculation
- The goal is weighted (entropy-weighted) similarity rather than unweighted similarity
  — then apply_weight_to_intensity or calculate_entropy_similarity should be used
  instead
- High-abundance noise or precursor ions are still present — clean_spectrum preprocessing
  (including removal of peaks above precursor_mz - precursor_ions_removal_da) should
  be run upstream first
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: ms_entropy
  role: Python package providing clean_spectrum and calculate_unweighted_entropy_similarity
    for validation of the merged-spectrum entropy similarity at ms2_tolerance_in_da
    = 0.05
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds the two [m/z, intensity] peak arrays and the merged spectrum AB for
    vectorized ion matching and intensity renormalization
- name: SciPy
  role: Its scipy.stats entropy function can be used to compute spectral entropy directly
    from normalized intensities, skipping the clean_spectrum step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:ce3fbbfe2beaa47abbd6c0ac9d5c36ff28409358166564064a790f85398a6fea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - NumPy
  - Python
  - SciPy
  - ms_entropy
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# msms-spectra-merging-and-renormalization

## Summary

Builds the 1:1 merged, weighted spectrum AB from two entropy-normalized MS/MS spectra by matching ions within 0.05 Da tolerance and renormalizing merged intensities to sum = 1, enabling computation of unweighted entropy similarity per equation (2).

## When to use

Use when two centroided MS/MS peak lists (m/z, intensity) must be combined into a merged spectrum AB to compute spectral entropy similarity, e.g. for library matching or compound identification scoring.

## When NOT to use

- Spectra have not been centroided and entropy-normalized first (intensities do not sum to 1) — merge before normalization invalidates the entropy calculation
- The goal is weighted (entropy-weighted) similarity rather than unweighted similarity — then apply_weight_to_intensity or calculate_entropy_similarity should be used instead
- High-abundance noise or precursor ions are still present — clean_spectrum preprocessing (including removal of peaks above precursor_mz - precursor_ions_removal_da) should be run upstream first

## Inputs

- Entropy-normalized MS/MS spectrum A as a numpy array of [m/z, intensity] peaks (intensities sum to 1)
- Entropy-normalized MS/MS spectrum B as a numpy array of [m/z, intensity] peaks (intensities sum to 1)
- MS2 matching tolerance in Da (0.05 Da per the workflow)

## Outputs

- Merged, weighted spectrum AB with intensities renormalized to sum = 1
- Shannon spectral entropies S_A, S_B, and S_AB
- similarity_score.json containing the unweighted entropy similarity (float in [0, 1]) plus S_A, S_B, S_AB

## How to apply

Start from two entropy-normalized spectra (all ion intensities adjusted to sum = 1) represented as numpy arrays of [m/z, intensity] peaks, with low-abundance ions <1% base peak already removed upstream. Match ions between spectra A and B within 0.05 Da tolerance to construct the merged, weighted spectrum AB, then renormalize AB intensities so the sum = 1. Compute the Shannon spectral entropies S_A, S_B, and S_AB from the normalized intensities (e.g. via scipy.stats.entropy or ms_entropy), and derive the unweighted entropy similarity as 1 - (2*S_AB - S_A - S_B)/ln(4) per equation (2). Merging must be 1:1 — each matched ion pair contributes one combined peak — because the entropy of the merged spectrum captures shared versus unique ions between the two spectra. Save the similarity score along with S_A, S_B, and S_AB to similarity_score.json, and cross-check the value against ms_entropy.calculate_unweighted_entropy_similarity at ms2_tolerance_in_da = 0.05.

## Related tools

- **ms_entropy** (Python package providing clean_spectrum and calculate_unweighted_entropy_similarity for validation of the merged-spectrum entropy similarity at ms2_tolerance_in_da = 0.05) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds the two [m/z, intensity] peak arrays and the merged spectrum AB for vectorized ion matching and intensity renormalization)
- **SciPy** (Its scipy.stats entropy function can be used to compute spectral entropy directly from normalized intensities, skipping the clean_spectrum step)

## Examples

```
import numpy as np, ms_entropy as me; sim = me.calculate_unweighted_entropy_similarity(peaks_a, peaks_b, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- similarity_score.json exists and the reported unweighted entropy similarity is a float in [0, 1]
- Reported similarity matches recomputation via ms_entropy.calculate_unweighted_entropy_similarity at ms2_tolerance_in_da = 0.05
- Merged spectrum AB intensities sum to 1 after renormalization
- Identical spectra should yield similarity = 1 (S_AB = S_A = S_B, so the numerator 2*S_AB - S_A - S_B = 0)
- S_A and S_B are non-negative Shannon entropies and 2*S_AB - S_A - S_B lies within [-ln(4), ln(4)]

## Limitations

- Low-abundance ions at <1% base peak intensity are removed before matching; spectra with dense low-intensity noise will behave differently if this filter is skipped
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values
- No single similarity threshold prevents false-positive discoveries; a score of 0.75 gave FDR < 10% only on 37,299 natural product spectra

## Evidence

- [methods] Unweighted entropy similarity formula per equation (2): [evidence span withheld — anchor status: not_found]
