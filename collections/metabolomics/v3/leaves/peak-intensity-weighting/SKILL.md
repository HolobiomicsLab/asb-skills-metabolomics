---
name: peak-intensity-weighting
description: Use when scoring similarity between two ESI-MS2 peak lists (m/z, intensity
  pairs) for compound identification against MassBank-style reference spectra, or
  when comparing exponent choices (m=0.5/n=2 vs m=0.6/n=3) at a 0.6 score threshold.
when_to_use_negative:
- Input spectra are EI-MS data — Stein and Scott's exponents (m=0.6, n=3) were derived
  for EI and the ESI-MS2-optimized m=0.5, n=2 may not transfer
- Input is already a similarity matrix or identification result table — the weighting
  step is redundant
- 'You need the merged-record identification precision gains: those require merging
  ESI-MS2 records across collision-induced dissociation conditions, not just re-weighting
  peaks'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NumPy
  role: Vectorized computation of the weighted peak values W_i = [Intensity_i]^0.5
    * [m/z_i]^2 and the weighted cosine correlation (dot product over vector norms)
- name: pandas
  role: Loading the unmerged MassBank ESI-MS2 query and reference spectra (3045 records
    of 679 synthetic drugs, January 2010 release) and assembling the per-exponent-pair
    metrics table
- name: SciPy
  role: Peak alignment/scoring support for matching query peaks to reference peaks
    on m/z
- name: MassBank3
  role: Serves the spectral reference library and REST API from which reference peak
    lists are retrieved
  repo: https://github.com/MassBank/MassBank3
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:8d7483afce5d47b33eafab225c089fdeaf2a9788f4f07666589c79e5ddb73046
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank3
  - NumPy
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# peak-intensity-weighting

## Summary

Implements MassBank's spectral-search similarity score: a weighted cosine correlation in which each peak i is weighted by W_i = [Intensity_i]^m * [m/z_i]^n, using the ESI-MS2-optimized exponents m = 0.5 and n = 2. These exponents differ from Stein and Scott's EI-MS values (m = 0.6, n = 3) and were tuned on 8785 ESI-MS2 spectra of ~700 authentic primary metabolites.

## When to use

Use when scoring similarity between two ESI-MS2 peak lists (m/z, intensity pairs) for compound identification against MassBank-style reference spectra, or when comparing exponent choices (m=0.5/n=2 vs m=0.6/n=3) at a 0.6 score threshold.

## When NOT to use

- Input spectra are EI-MS data — Stein and Scott's exponents (m=0.6, n=3) were derived for EI and the ESI-MS2-optimized m=0.5, n=2 may not transfer
- Input is already a similarity matrix or identification result table — the weighting step is redundant
- You need the merged-record identification precision gains: those require merging ESI-MS2 records across collision-induced dissociation conditions, not just re-weighting peaks

## Inputs

- Query ESI-MS2 peak list (array of (m/z, intensity) pairs)
- Reference ESI-MS2 peak list from a MassBank-style record (array of (m/z, intensity) pairs)
- Optional: exponent pair (m, n) — default m=0.5, n=2 for ESI-MS2; m=0.6, n=3 for Stein-Scott EI-MS baseline

## Outputs

- Scalar similarity score in [0, 1] per query-reference pair
- Optional: comparison table (e.g. exponent_comparison.csv) with precision, recall, and F-value per exponent pair at threshold 0.6

## How to apply

Load the query and reference peak lists as arrays of (m/z, intensity) pairs and compute the weight W_i = [Intensity_i]^0.5 * [m/z_i]^2 for every peak in both spectra (Eqn 4, ESI-MS2-optimized exponents). Align query peaks to reference peaks on m/z so matched peaks form paired weighted-vector components and unmatched peaks contribute zero. Compute the cosine correlation of the two weighted vectors (dot product divided by the product of vector norms), yielding a scalar in [0, 1]; classify identifications at MassBank's similarity score threshold of 0.6. If benchmarking against Stein-Scott's EI-derived weighting, re-score the same pairs with m = 0.6 and n = 3 and compare precision, recall, and F-value per exponent pair.

## Related tools

- **NumPy** (Vectorized computation of the weighted peak values W_i = [Intensity_i]^0.5 * [m/z_i]^2 and the weighted cosine correlation (dot product over vector norms))
- **pandas** (Loading the unmerged MassBank ESI-MS2 query and reference spectra (3045 records of 679 synthetic drugs, January 2010 release) and assembling the per-exponent-pair metrics table)
- **SciPy** (Peak alignment/scoring support for matching query peaks to reference peaks on m/z)
- **MassBank3** (Serves the spectral reference library and REST API from which reference peak lists are retrieved) — https://github.com/MassBank/MassBank3

## Examples

```
W = (intensities**0.5) * (mz**2); score = np.dot(Wq, Wr)/(np.linalg.norm(Wq)*np.linalg.norm(Wr))
```

## Evaluation signals

- Identical query and reference peak lists must yield a similarity score of exactly 1.0
- All computed scores must fall within [0, 1]
- The score scale must be consistent with MassBank's reported identification threshold of 0.6
- Precision values at threshold 0.6 with m=0.5/n=2 must be consistent with the paper's baseline: 0.222 for TS=[QqQ] and 0.276 for TS=[QqTOF]
- Comparison table contains one row per exponent pair with precision, recall, and F-value columns (e.g. exponent_comparison.csv)

## Limitations

- Exponents m=0.5, n=2 were optimized specifically for ESI-MS2 data and differ from Stein and Scott's EI-MS exponents (m=0.6, n=3); unmerged-search performance at threshold 0.6 is modest (precision 0.222–0.276, recall 0.292–0.327, F-value 0.264–0.284)
- Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions — not exponent tuning alone — improved precision by 21–23% at a similarity score of 0.6
- The MassBank-web repository containing the original MassBank software is deprecated in favor of MassBank3 (web app) and MassBank-cli-tools (validator/CLI tools)

## Evidence

- [abstract] Weighted cosine correlation with ESI-MS2 exponents: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [results] Identification threshold context: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [readme] Current software location: "This repo is deprecated. Please do not use it! The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3."
