---
name: esi-ms2-spectral-search
description: Use when scoring query ESI-MS2 peak lists (m/z, intensity pairs) against
  reference spectra from MassBank and a compound identification decision at a similarity
  score threshold (0.6) is required.
when_to_use_negative:
- The spectra are EI-MS rather than ESI-MS2 — Stein and Scott's EI-derived exponents
  (m=0.6, n=3) apply to that regime, so blindly using m=0.5/n=2 is out of scope
- 'You need merged multi-condition spectra: MassBank showed data merging under different
  collision-induced dissociation conditions improves precision by 21-23% at a score
  of 0.6, which is a different pipeline than unmerged per-record search'
- The input is already a similarity matrix or identification table rather than raw
  peak lists
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: NumPy
  role: Implements the weighted cosine correlation score (weighting exponents applied
    as vectorized array operations on peak lists)
- name: pandas
  role: Loads and manages the MassBank query/reference spectra records for batch scoring
- name: SciPy
  role: Supports peak alignment/scoring computations in the sensitivity analysis workflow
- name: MassBank3
  role: Hosts the spectral reference library whose records serve as reference peak
    lists for spectral search
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Validator and command line tools for MassBank record data used as search input
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:6ef8ed6fd8e3482faecea584e1067bf5f4fe43604c4e65a07761547d939f6dbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank-cli-tools
  - MassBank3
  - NumPy
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# esi-ms2-spectral-search

## Summary

Implements MassBank's spectral search similarity score: a weighted cosine correlation in which each peak is weighted by W_i = [Intensity_i]^m * [m/z_i]^n, with exponents optimized to ESI-MS2 data as m=0.5 and n=2, returning a scalar similarity score in [0, 1] compared against MassBank's identification threshold of 0.6.

## When to use

Use when scoring query ESI-MS2 peak lists (m/z, intensity pairs) against reference spectra from MassBank and a compound identification decision at a similarity score threshold (0.6) is required.

## When NOT to use

- The spectra are EI-MS rather than ESI-MS2 — Stein and Scott's EI-derived exponents (m=0.6, n=3) apply to that regime, so blindly using m=0.5/n=2 is out of scope
- You need merged multi-condition spectra: MassBank showed data merging under different collision-induced dissociation conditions improves precision by 21-23% at a score of 0.6, which is a different pipeline than unmerged per-record search
- The input is already a similarity matrix or identification table rather than raw peak lists

## Inputs

- Query ESI-MS2 peak list as (m/z, intensity) pairs
- Reference ESI-MS2 peak list from a MassBank record as (m/z, intensity) pairs
- Optional: MassBank record collection (e.g. 3045 ESI-MS2 records of 679 synthetic drugs, January 2010 release)

## Outputs

- Scalar weighted cosine correlation similarity score per query-reference pair, in [0, 1]
- Identification calls per pair at similarity score threshold 0.6
- Optional comparison table (e.g. exponent_comparison.csv) with precision, recall and F-value per exponent pair (m=0.5/n=2 vs m=0.6/n=3)

## How to apply

Load the query and reference peak lists as arrays of (m/z, intensity) pairs and compute the weight W_i = [Intensity_i]^0.5 * [m/z_i]^2 for every peak in both spectra (Eqn 4, ESI-MS2-optimized exponents m=0.5, n=2). Align query peaks to reference peaks on m/z so matched peaks form paired components of the weighted vectors and unmatched peaks contribute zero. Compute the weighted cosine correlation — dot product of the weighted vectors divided by the product of their norms — yielding a scalar in [0, 1]; classify an identification when the score meets the MassBank threshold of 0.6. For comparison against the EI-MS baseline, re-run the score with Stein and Scott's exponents m=0.6, n=3, since these were optimized for EI rather than ESI-MS2 data; MassBank's exponents were determined by analyzing 8785 ESI-MS2 spectra of ca. 700 authentic compounds of primary metabolites. Implement the score as a single callable function (NumPy) and batch-score all query-reference pairs (e.g. pandas over MassBank records such as the 3045 ESI-MS2 records of 679 synthetic drugs from the January 2010 release), computing precision, recall and F-value at the 0.6 threshold to validate behavior.

## Related tools

- **NumPy** (Implements the weighted cosine correlation score (weighting exponents applied as vectorized array operations on peak lists))
- **pandas** (Loads and manages the MassBank query/reference spectra records for batch scoring)
- **SciPy** (Supports peak alignment/scoring computations in the sensitivity analysis workflow)
- **MassBank3** (Hosts the spectral reference library whose records serve as reference peak lists for spectral search) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Validator and command line tools for MassBank record data used as search input) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
import numpy as np; W = lambda mz, it, m=0.5, n=2: it**m * mz**n; score = W(q_mz,q_int).dot(W(r_mz,r_int)) / (np.linalg.norm(W(q_mz,q_int)) * np.linalg.norm(W(r_mz,r_int)))
```

## Evaluation signals

- Identical query and reference peak lists must yield a similarity score of exactly 1.0
- All computed scores must fall within [0, 1]
- The score scale must be consistent with MassBank's reported identification threshold of 0.6
- Precision at threshold 0.6 with m=0.5/n=2 must be consistent with the paper's reported values (e.g. precision 0.222 / recall 0.327 / F 0.264 for TS=[QqQ]; 0.276 / 0.292 / 0.284 for TS=[QqTOF] on the unmerged search)
- An output table like exponent_comparison.csv must contain one row per exponent pair with precision, recall, and F-value columns

## Limitations

- The exponents m=0.5, n=2 were optimized for ESI-MS2 data of primary metabolites (8785 spectra of ca. 700 authentic compounds); they differ from Stein and Scott's EI-MS exponents (m=0.6, n=3) and are not transferable to EI data
- On the unmerged search at threshold 0.6, precision is low (0.222 for QqQ, 0.276 for QqTOF); merging ESI-MS2 data recorded under different collision-induced dissociation conditions improved precision by 21-23%, so unmerged scores should be interpreted cautiously
- The MassBank-web repository is deprecated — the web app is maintained in MassBank3 and command line/validator tools in MassBank-cli-tools

## Evidence

- [abstract] weighted cosine correlation with ESI-MS2 exponents: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] threshold 0.6 and merging effect: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [readme] MassBank3 as the reference library software: [evidence span withheld — anchor status: not_found]
- [readme] deprecated MassBank-web repo pointer: "The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools"
