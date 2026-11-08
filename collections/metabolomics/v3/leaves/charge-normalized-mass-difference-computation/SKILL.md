---
name: charge-normalized-mass-difference-computation
description: Use when comparing a modified query spectrum to an unmodified library
  spectrum and the precursor m/z difference must be converted into a neutral mass
  delta (charge-normalized) to derive candidate modification-induced peak shifts.
when_to_use_negative:
- When performing a standard (closed) search where query and library precursor masses
  are expected to match within tolerance — ANN-SoLo first identifies unmodified peptides
  in the cascade search before the open search that needs this shift
- When the precursor charge state is unknown or misassigned, since the normalization
  to a neutral mass delta depends critically on the charge (differing charges between
  query and library must be explicitly accounted for)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Open modification spectral library search engine that computes the charge-normalized
    precursor mass difference as part of candidate selection and uses it to drive
    the shifted dot product score
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for ANN-SoLo in which the precursor mass difference
    parsing and charge normalization are carried out
  repo: https://github.com/bittremieux/ANN-SoLo
- name: C++
  role: Compiled component of ANN-SoLo used for the performance-critical spectral
    scoring that consumes the normalized mass shift
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:6f8a3fabd6b801a84a7f993b30a466f3054463589d819e138c813356a8e378b8
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# charge-normalized-mass-difference-computation

## Summary

Computes the precursor mass difference between a query and library spectrum normalized by precursor charge, providing the shift used by the shifted dot product to match modified spectra to their unmodified library counterparts in open modification spectral library searching (ANN-SoLo).

## When to use

Use when comparing a modified query spectrum to an unmodified library spectrum and the precursor m/z difference must be converted into a neutral mass delta (charge-normalized) to derive candidate modification-induced peak shifts.

## When NOT to use

- When performing a standard (closed) search where query and library precursor masses are expected to match within tolerance — ANN-SoLo first identifies unmodified peptides in the cascade search before the open search that needs this shift
- When the precursor charge state is unknown or misassigned, since the normalization to a neutral mass delta depends critically on the charge (differing charges between query and library must be explicitly accounted for)

## Inputs

- Query spectrum peak lists with precursor m/z and charge (e.g., MGF files from MassIVE accession MSV000078492 or PRIDE PXD001468/PXD009861)
- Library spectrum entries with precursor m/z and charge (e.g., NIST human HCD spectral library v2016/09/12, MassIVE-KB v2017/11/27, TripleTOF yeast library from Selevsek et al.)

## Outputs

- Charge-normalized precursor mass difference (neutral modification mass delta) per query-library candidate pair
- Shift value consumed by the shifted dot product score for each candidate SSM
- Subgroup assignments (by charge-normalized precursor mass difference) used for FDR calculation
- Verification artifacts such as shifted_dot_product_verification.json with computed scores per pair and the recovered shift

## How to apply

Parse the query and library spectra (e.g., MGF from MassIVE/PXDA datasets) into peak lists with their precursor m/z and charge values. Compute the precursor mass difference and normalize it according to the precursor charge — for differing charges between query and library, normalize by multiplying the m/z difference by the query charge to obtain the neutral modification mass. This charge-normalized delta is then used by the shifted dot product: in a linear pass through both spectra, potential peak pairs are enumerated both unshifted and shifted by this delta, scored by the product of the two peak intensities, with fully penalized shifted matches lacking an annotation to limit spurious matches. The same charge-normalized mass difference also defines the subgrouping key for FDR estimation, since SSMs are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra (filtering only subgroups containing at least 20 SSMs).

## Related tools

- **ANN-SoLo** (Open modification spectral library search engine that computes the charge-normalized precursor mass difference as part of candidate selection and uses it to drive the shifted dot product score) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for ANN-SoLo in which the precursor mass difference parsing and charge normalization are carried out) — https://github.com/bittremieux/ANN-SoLo
- **C++** (Compiled component of ANN-SoLo used for the performance-critical spectral scoring that consumes the normalized mass shift) — https://github.com/bittremieux/ANN-SoLo

## Examples

```
mass_shift = (precursor_mz_query - precursor_mz_library) * charge_query  # charge-normalized neutral delta for the shifted dot product
```

## Evaluation signals

- The normalized delta must equal (precursor m/z difference) × charge, i.e., represent the neutral modification mass, not the m/z-space difference
- When query and library spectra have different charges, the normalization must account for both charges so the same modification yields the same delta
- Synthetic modified/unmodified spectrum pairs (base spectrum plus a copy with peaks shifted by a known mass delta) should recover the expected shift and score the true modified counterpart higher than decoy pairs
- Grouping SSMs by the computed charge-normalized precursor mass difference must yield subgroups before FDR filtering is applied only to subgroups with at least 20 SSMs
- On the iPRG2012 data set, the downstream open search using these shifts should yield ~25% more identifications than a standard search, with additional identifications corresponding to modified peptides

## Limitations

- The actual FDR might be underestimated when too-small subgroups (fewer than 20 SSMs) of charge-normalized mass differences are used
- Accurate FDR estimation for open searches keyed on mass differences remains an unresolved research question
- Non-metric space indexing to retrieve candidates directly by the shifted dot product similarity has only been mentioned as a potential, untested approach

## Evidence

- [full_text] Parse the query and library spectra into peak lists and compute the precursor mass difference, normalized according to the precursor charge.: [evidence span withheld — anchor status: not_found]
- [methods] Subgrouping by charge-normalized mass difference: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [intro] Shifted dot product rationale: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts"
- [full_text] Verification against synthetic pairs: [evidence span withheld — anchor status: not_found]
