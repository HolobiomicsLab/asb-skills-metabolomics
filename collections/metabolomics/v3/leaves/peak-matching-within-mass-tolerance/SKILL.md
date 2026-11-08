---
name: peak-matching-within-mass-tolerance
description: Use when two entropy-normalized MS/MS spectra (intensity sums = 1, low-abundance
  ions already removed) must be ion-matched within 0.05 Da to form a merged spectrum
  for Shannon-entropy similarity scoring.
when_to_use_negative:
- Spectra are still raw/profile or uncleaned — merge/remove low-abundance ions (<1%
  base peak) and precursor-region ions (m/z > precursor_mz - 1.6) first via clean_spectrum.
- You need weighted entropy similarity (entropy-weighted intensities) rather than
  the unweighted similarity of equation (2) — use calculate_entropy_similarity / apply_weight_to_intensity
  instead.
- You need to search one query against a large library rather than compare a single
  spectrum pair — use FlashEntropySearch or DynamicEntropySearch indexing instead
  of pairwise matching.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms_entropy
  role: Provides clean_spectrum() preprocessing and calculate_unweighted_entropy_similarity()
    with ms2_tolerance_in_da = 0.05 to perform/verify the ion matching and merged-spectrum
    similarity computation.
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds the two spectra as [m/z, intensity] numpy arrays used for the 0.05 Da
    ion-matching and merged-spectrum construction.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:aa626220725cb820b821a4141113c70cfb7f4876a5c983e7a9d56fe38183461d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - NumPy
  - Python
  - ms_entropy
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# peak-matching-within-mass-tolerance

## Summary

Matches ions between two MS/MS spectra within a 0.05 Da m/z tolerance to build the merged, weighted spectrum AB, which is the core intermediate for computing unweighted entropy similarity (equation 2).

## When to use

Use when two entropy-normalized MS/MS spectra (intensity sums = 1, low-abundance ions already removed) must be ion-matched within 0.05 Da to form a merged spectrum for Shannon-entropy similarity scoring.

## When NOT to use

- Spectra are still raw/profile or uncleaned — merge/remove low-abundance ions (<1% base peak) and precursor-region ions (m/z > precursor_mz - 1.6) first via clean_spectrum.
- You need weighted entropy similarity (entropy-weighted intensities) rather than the unweighted similarity of equation (2) — use calculate_entropy_similarity / apply_weight_to_intensity instead.
- You need to search one query against a large library rather than compare a single spectrum pair — use FlashEntropySearch or DynamicEntropySearch indexing instead of pairwise matching.

## Inputs

- Two entropy-normalized MS/MS peak lists (numpy arrays of [m/z, intensity] with intensity sums = 1)
- MS2 peak-matching tolerance (0.05 Da as used in the paper)

## Outputs

- Merged, weighted spectrum AB with intensities renormalized to sum = 1
- Shannon entropies S_A, S_B, S_AB (floats)
- Unweighted entropy similarity score (float in [0, 1]), e.g. saved to similarity_score.json

## How to apply

Start from the two cleaned, entropy-normalized spectra A and B as numpy arrays of [m/z, intensity] peaks (peaks below 1% base peak intensity and ions above precursor_mz - 1.6 should already be removed by upstream cleaning, e.g. ms_entropy.clean_spectrum). Pair each ion of spectrum A with the nearest ion of spectrum B within 0.05 Da; when peaks merge, combine their intensities into the weighted merged spectrum AB, then renormalize AB intensities so they sum to 1. Compute Shannon spectral entropies S_A, S_B, and S_AB from the normalized intensities, then evaluate similarity = 1 - (2*S_AB - S_A - S_B)/ln(4) per equation (2). Rationale: matching within 0.05 Da keeps genuinely shared fragments aligned while leaving unmatched noise ions distinct, which is why entropy similarity stays robust against noise whereas dot product similarity degrades. Cross-check the result with ms_entropy.calculate_unweighted_entropy_similarity at ms2_tolerance_in_da = 0.05.

## Related tools

- **ms_entropy** (Provides clean_spectrum() preprocessing and calculate_unweighted_entropy_similarity() with ms2_tolerance_in_da = 0.05 to perform/verify the ion matching and merged-spectrum similarity computation.) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds the two spectra as [m/z, intensity] numpy arrays used for the 0.05 Da ion-matching and merged-spectrum construction.)

## Examples

```
import numpy as np, ms_entropy as me; sim = me.calculate_unweighted_entropy_similarity(peaks_a, peaks_b, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- Output file (e.g. similarity_score.json) exists and contains S_A, S_B, S_AB and a similarity score that is a float in [0, 1].
- Score recomputed with ms_entropy.calculate_unweighted_entropy_similarity(..., ms2_tolerance_in_da=0.05) matches the manually computed value.
- Merged spectrum AB intensities renormalize to sum = 1 before entropy calculation.
- Identical spectra yield similarity 1.0 (2*S_AB - S_A - S_B = 0); completely disjoint peaks approach 0.

## Limitations

- Low-abundance ions at <1% base peak intensity must be removed before matching, and ions are matched at 0.05 Da tolerance — different tolerances change the merged spectrum and score.
- Even with entropy similarity, no single similarity threshold prevents false-positive discoveries; a score threshold of 0.75 gave FDR < 10% only on the natural-product benchmark.
- Power transformations with w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values, so unweighted (or properly weighted) intensities should not be arbitrarily transformed.

## Evidence
