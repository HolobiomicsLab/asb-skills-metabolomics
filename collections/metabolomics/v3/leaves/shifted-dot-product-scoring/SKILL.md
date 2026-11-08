---
name: shifted-dot-product-scoring
description: Use when performing open modification spectral library search on tandem
  MS spectra where a query spectrum has a nonzero charge-normalized precursor mass
  difference from its library candidate and modified peptide spectra must be matched
  to unmodified library spectra.
when_to_use_negative:
- First (unmodified) level of a cascade search, where the standard dot product on
  unshifted peak pairs is appropriate and the query/library precursor masses agree
- Candidate selection (precursor mass filtering / ANN retrieval), which precedes candidate
  ranking — the shifted dot product is only the spectrum–spectrum match score used
  during ranking
- Sequence database searching scenarios where fragment-shift matching against theoretical
  spectra is not the scoring model
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Search engine in which the shifted dot product is the candidate-ranking SSM
    score for matching modified query spectra to unmodified library spectra
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for parsing spectra, computing peak-pair intensity
    products, and greedy summation of the score
- name: NumPy
  role: Vectorized computation of peak-pair intensity products over enumerated shifted
    and unshifted peak matches
- name: C++
  role: Performance-critical implementation of the shifted dot product enumeration
    pass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:ee0f1216abc2ad371df5f1327e40046516e0b238d6065bd5fb242274f70092f2
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# shifted-dot-product-scoring

## Summary

Scoring function from ANN-SoLo that matches a modified query MS/MS spectrum to its unmodified library counterpart by enumerating peak pairs both unshifted and shifted by the charge-normalized precursor mass difference, scoring pairs by intensity product and summing them greedily. Replacing the standard dot product with the shifted dot product yielded a further 21% increase in identifications of modified spectra.

## When to use

Use when performing open modification spectral library search on tandem MS spectra where a query spectrum has a nonzero charge-normalized precursor mass difference from its library candidate and modified peptide spectra must be matched to unmodified library spectra.

## When NOT to use

- First (unmodified) level of a cascade search, where the standard dot product on unshifted peak pairs is appropriate and the query/library precursor masses agree
- Candidate selection (precursor mass filtering / ANN retrieval), which precedes candidate ranking — the shifted dot product is only the spectrum–spectrum match score used during ranking
- Sequence database searching scenarios where fragment-shift matching against theoretical spectra is not the scoring model

## Inputs

- Query MS/MS spectra as peak lists with precursor m/z and charge (e.g. mzML/MGF query files)
- Library spectra peak lists with precursor m/z (e.g. NIST human HCD spectral library, MassIVE-KB library)
- Charge-normalized precursor mass difference between query and library candidate

## Outputs

- Shifted dot product score per query–library candidate spectrum–spectrum match
- Shift recovered per matched pair (verification runs: shifted_dot_product_verification.json)

## How to apply

Parse the query and library spectra into peak lists and compute the precursor mass difference normalized by precursor charge. In a linear pass through both spectra, enumerate all potential peak pairs — unshifted and shifted by the charge-normalized precursor mass difference (accounting for different charges) — scoring each pair by the product of the two peak intensities; apply a full penalty to shifted matches that lack an annotation to limit spurious matches. Sort peak matches by intensity product and sum them greedily, enforcing that no single peak in either spectrum is matched more than once, to obtain the shifted dot product score. Validate by generating synthetic modified/unmodified spectrum pairs (a base spectrum plus a copy with peaks shifted by a known mass delta) and confirming the implementation recovers the expected shift and scores the true modified counterpart higher than decoy pairs.

## Related tools

- **ANN-SoLo** (Search engine in which the shifted dot product is the candidate-ranking SSM score for matching modified query spectra to unmodified library spectra) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for parsing spectra, computing peak-pair intensity products, and greedy summation of the score)
- **NumPy** (Vectorized computation of peak-pair intensity products over enumerated shifted and unshifted peak matches)
- **C++** (Performance-critical implementation of the shifted dot product enumeration pass)

## Examples

```
python verify_shifted_dot_product.py --query query.mgf --library nist_human_hcd.splib --output shifted_dot_product_verification.json
```

## Evaluation signals

- On synthetic modified/unmodified spectrum pairs (base spectrum plus copy with peaks shifted by a known mass delta), the implementation recovers the expected shift exactly
- The true modified counterpart is scored higher than decoy pairs on synthetic spectra
- Verification output schema contains computed scores per pair and the shift recovered, written to shifted_dot_product_validation format (shifted_dot_product_verification.json)
- No single peak in either spectrum is matched more than once after greedy intensity-product summation
- Fully penalized shifted matches without annotations are suppressed, keeping spurious shifted matches limited

## Limitations

- Shifted matches without annotations are fully penalized; spurious shifted matches are only limited by this penalty, not eliminated
- Direct retrieval of candidates using the shifted dot product (non-metric space indexing) was only mentioned as a potential, untested approach
- Like the surrounding ANN-SoLo search, FDR estimates downstream may be underestimated when subgroups (grouped by charge-normalized precursor mass difference) contain fewer than 20 SSMs

## Evidence

- [intro] the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [readme] the shifted dot product score to sensitively match modified spectra to their unmodified counterpart.: "combined with a cascade search strategy to maximize the number of identified unmodified and modified spectra while strictly controlling the false discovery rate and the shifted dot product score to"
- [results] A further 21% increase in identifications is achieved by using the shifted dot product instead of the standard dot product to score the spectral similarity of modified spectra: [evidence span withheld — anchor status: not_found]
- [discussion] non-metric space indexing can potentially be used to retrieve candidates based on the shifted dot product similarity directly [10].: "non-metric space indexing can potentially be used to retrieve candidates based on the shifted dot product similarity directly [10]."
