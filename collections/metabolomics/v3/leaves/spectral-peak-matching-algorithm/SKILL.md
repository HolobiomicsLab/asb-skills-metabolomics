---
name: spectral-peak-matching-algorithm
description: Use when scoring a modified query spectrum against its unmodified library
  counterpart in open modification spectral library searching, where fragment ion
  peak shifts equal to the precursor mass delta must be accounted for during matching.
when_to_use_negative:
- Standard (non-open) library searching of unmodified spectra, where the unshifted
  dot product is sufficient and the cascade search already resolves unmodified peptides
  at its first level
- Candidate retrieval itself — the shifted dot product is only used for candidate
  ranking; direct retrieval of candidates via non-metric space indexing on this score
  was only mentioned as an untested future approach
- Sequence database searching (e.g. MSFragger-style fragment indexing), where peptide
  sequences rather than spectral library spectra are the reference
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Search engine that implements the shifted dot product as its SSM score during
    the candidate ranking step of the cascade search
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for ANN-SoLo, used to write the peak-pair enumeration
    and greedy matching logic
- name: C++
  role: Implementation language for performance-critical parts of ANN-SoLo's scoring
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:ce07cb20ac56fff759201119757f16cda6290b0dbb2a36c7c5afe549b7e90ecd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - C++
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# spectral-peak-matching-algorithm

## Summary

The shifted dot product is ANN-SoLo's spectrum–spectrum matching score that matches modified query spectra to unmodified library spectra by explicitly scoring peak pairs both unshifted and shifted by the charge-normalized precursor mass difference, yielding a 21% identification increase over the standard dot product for modified spectra.

## When to use

Use when scoring a modified query spectrum against its unmodified library counterpart in open modification spectral library searching, where fragment ion peak shifts equal to the precursor mass delta must be accounted for during matching.

## When NOT to use

- Standard (non-open) library searching of unmodified spectra, where the unshifted dot product is sufficient and the cascade search already resolves unmodified peptides at its first level
- Candidate retrieval itself — the shifted dot product is only used for candidate ranking; direct retrieval of candidates via non-metric space indexing on this score was only mentioned as an untested future approach
- Sequence database searching (e.g. MSFragger-style fragment indexing), where peptide sequences rather than spectral library spectra are the reference

## Inputs

- Preprocessed query spectrum peak list (peaks below 1% of the most intense peak removed, restricted to 50 most intense peaks, spectra with fewer than 10 peaks or mass range <250 Da discarded)
- Preprocessed library spectrum peak list (e.g. from the NIST human HCD library or MassIVE-KB library)
- Charge-normalized precursor mass difference between the query spectrum and the candidate library spectrum

## Outputs

- Shifted dot product SSM score per query–candidate library spectrum pair
- Spectrum–spectrum matches (SSMs) for subsequent FDR filtering, subgrouped by charge-normalized precursor mass difference
- Verification results per synthetic pair, including computed scores and the recovered mass shift (e.g. written to shifted_dot_product_verification.json)

## How to apply

Parse both spectra into peak lists and compute the precursor mass difference normalized by precursor charge. In a linear pass over both spectra, enumerate all candidate peak pairs: unshifted matches and matches shifted by the charge-normalized precursor mass difference (also considering different charges), scoring each pair as the product of the two peak intensities; shifted matches without an annotation are fully penalized to limit spurious matches. Sort all peak matches by intensity product and sum them greedily, enforcing that no single peak in either spectrum is matched more than once, to obtain the shifted dot product score. This score is applied during ANN-SoLo's candidate ranking step after ANN-based candidate selection, and SSMs are then grouped by charge-normalized precursor mass difference for FDR filtering. Verify the implementation on synthetic modified/unmodified spectrum pairs: a base spectrum plus a copy with peaks shifted by a known mass delta should recover the expected shift and score the true modified counterpart above decoys.

## Related tools

- **ANN-SoLo** (Search engine that implements the shifted dot product as its SSM score during the candidate ranking step of the cascade search) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for ANN-SoLo, used to write the peak-pair enumeration and greedy matching logic)
- **C++** (Implementation language for performance-critical parts of ANN-SoLo's scoring)

## Evaluation signals

- On synthetic modified/unmodified spectrum pairs (base spectrum plus a copy with peaks shifted by a known mass delta), the implementation recovers the expected shift exactly
- The true modified counterpart receives a higher shifted dot product score than decoy pairs in the synthetic verification
- No peak in either spectrum is matched more than once in the greedy sum (each peak appears in at most one matched pair)
- Fully penalized shifted matches (shifted pairs without annotation) contribute nothing to the score, limiting spurious matches
- Using the shifted dot product instead of the standard dot product yields a ~21% increase in identifications on modified spectra

## Limitations

- Shifted matches without an annotation are fully penalized, which limits spurious matches but means unexplained shifted peak pairs carry no score
- The score requires the charge-normalized precursor mass difference, so the matching shift is restricted to the observed precursor delta (and its multiples for different charges), not arbitrary peak shifts
- Subgroup FDR estimation for SSMs grouped by charge-normalized mass difference may be underestimated when subgroups are too small
- Accurate FDR estimation for open searches remains an unresolved research question

## Evidence

- [intro] the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [methods] they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [readme] the shifted dot product score to sensitively match modified spectra to their unmodified counterpart: "the shifted dot product score to sensitively match modified spectra to their unmodified counterpart"
- [methods] a candidate ranking step during which, for each candidate spectrum, a spectrum–spectrum match (SSM) score is calculated: "a candidate ranking step during which, for each candidate spectrum, a spectrum–spectrum match (SSM) score is calculated"
