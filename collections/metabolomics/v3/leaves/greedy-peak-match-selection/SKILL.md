---
name: greedy-peak-match-selection
description: Use when scoring a modified query spectrum against an unmodified library
  spectrum where the precursor mass difference implies fragment peak shifts and each
  peak must be counted at most once in the match.
when_to_use_negative:
- Query and library spectra are expected to be unmodified, so no charge-normalized
  mass shift needs to be accounted for — the standard dot product suffices
- Spectra have not been preprocessed (noise peaks below 1% of the base peak still
  present, more than 50 peaks, or fewer than 10 peaks / mass range below 250 Da),
  violating the assumptions of the matching pass
- The task is candidate selection rather than candidate ranking — the ANN index, not
  the shifted dot product, is the appropriate stage for narrowing down library spectra
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Search engine whose candidate ranking step computes the shifted dot product
    score using this greedy peak-match selection
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for the linear peak-pair enumeration and greedy summation
- name: NumPy
  role: Numerical handling of peak lists and intensity-product scoring
- name: C++
  role: Performance-critical implementation of the scoring used by ANN-SoLo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:97c1a7044e9523e556bd8336618c816ec9b0aac2646281f5b2b66de1035ed719
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - C++
  - NumPy
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# greedy-peak-match-selection

## Summary

Scoring procedure within the shifted dot product that ranks all candidate unshifted and shifted peak pairs by intensity product and greedily sums the best matches while enforcing that no single peak in either the query or library spectrum is matched more than once. This yields a sensitive spectrum–spectrum match (SSM) score for modified query spectra against unmodified library counterparts.

## When to use

Use when scoring a modified query spectrum against an unmodified library spectrum where the precursor mass difference implies fragment peak shifts and each peak must be counted at most once in the match.

## When NOT to use

- Query and library spectra are expected to be unmodified, so no charge-normalized mass shift needs to be accounted for — the standard dot product suffices
- Spectra have not been preprocessed (noise peaks below 1% of the base peak still present, more than 50 peaks, or fewer than 10 peaks / mass range below 250 Da), violating the assumptions of the matching pass
- The task is candidate selection rather than candidate ranking — the ANN index, not the shifted dot product, is the appropriate stage for narrowing down library spectra

## Inputs

- Preprocessed peak lists for the query spectrum and candidate library spectrum (noise peaks below 1% of the base peak removed, at most 50 most intense peaks retained)
- Precursor m/z and charge of the query spectrum and of the library spectrum, used to compute the charge-normalized precursor mass difference

## Outputs

- Shifted dot product score per query–library spectrum pair (used for candidate ranking)
- Set of matched peak pairs annotated as unshifted or shifted by the charge-normalized mass difference
- Verification record of computed scores and recovered shifts (e.g., shifted_dot_product_verification.json)

## How to apply

Parse the query and library spectra into peak lists (after preprocessing: remove noise peaks below 1% of the most intense peak, keep at most the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range under 250 Da). Compute the charge-normalized precursor mass difference between the query and library spectra. In a linear pass, enumerate all potential peak pairs, both unshifted and shifted by the charge-normalized precursor mass difference (accounting for different charges), scoring each pair by the product of the two peak intensities; shifted matches without an annotation are fully penalized to limit spurious matches. Sort the peak matches by intensity product and sum them greedily, ensuring no single peak in either spectrum is matched more than once; the resulting sum is the shifted dot product score used for candidate ranking in ANN-SoLo's cascade search.

## Related tools

- **ANN-SoLo** (Search engine whose candidate ranking step computes the shifted dot product score using this greedy peak-match selection) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for the linear peak-pair enumeration and greedy summation)
- **NumPy** (Numerical handling of peak lists and intensity-product scoring)
- **C++** (Performance-critical implementation of the scoring used by ANN-SoLo)

## Examples

```
pairs.sort(key=lambda p: -p.intensity_product); used=set(); score=sum(m.intensity_product for m in pairs if m.q not in used and m.l not in used and not used.update((m.q, m.l)))
```

## Evaluation signals

- Generate synthetic modified/unmodified spectrum pairs (a base spectrum plus a copy with peaks shifted by a known mass delta) and verify the implementation recovers the expected shift
- Verify the true modified counterpart scores higher than decoy spectrum pairs under the greedy match
- Check the invariant that no single peak in either spectrum appears in more than one accepted match
- Confirm shifted matches lacking an annotation are fully penalized, limiting spurious matches
- Compare identifications against the standard dot product: the shifted dot product should yield a further ~21% increase in identifications on modified spectra

## Limitations

- Shifted peak matches without an annotation are fully penalized; this suppresses spurious matches but can penalize legitimate unexplained shifted peaks
- Scoring is restricted to preprocessed spectra (≤ 50 most intense peaks, ≥ 10 peaks, mass range ≥ 250 Da); spectra outside these bounds are discarded before matching
- Accurate FDR estimation for open searches remains an open research question, so scores from this matching step must be interpreted with care when filtering at 1% FDR

## Evidence

- [intro] the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [methods] a candidate ranking step during which, for each candidate spectrum, a spectrum–spectrum match (SSM) score is calculated: "a candidate ranking step during which, for each candidate spectrum, a spectrum–spectrum match (SSM) score is calculated"
- [readme] the shifted dot product score to sensitively match modified spectra to their unmodified counterpart: "the shifted dot product score to sensitively match modified spectra to their unmodified counterpart"
