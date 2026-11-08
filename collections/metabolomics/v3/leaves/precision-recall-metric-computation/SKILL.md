---
name: precision-recall-metric-computation
description: 'Use when evaluating spectral-search identification performance: you
  have query and reference peak lists (e.g. QqQ vs QqTOF ESI-MS2 spectra), per-pair
  similarity scores, and need precision/recall/F at a fixed score cutoff such as 0.6.'
when_to_use_negative:
- The reference spectra are already merged across collision-induced dissociation conditions
  — the merged search is a separate analysis (paper reports merged data improves precision
  by 21-23% at score 0.6), so unmerged-baseline metrics would not apply directly.
- The input is EI mass spectra intended for the Stein-Scott regime; the ESI-MS2-optimized
  exponents m=0.5/n=2 are not the appropriate weights there.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: pandas
  role: Loads the MassBank ESI-MS2 query and target peak lists and assembles the metric
    comparison tables (similarity_search_metrics.csv, exponent_comparison.csv)
- name: NumPy
  role: Implements the weighted cosine correlation similarity score applying peak-intensity
    exponent m and m/z exponent n as spectral weights
- name: MassBank-cli-tools
  role: MassBank command-line tools referenced for the spectral search/validation
    workflow
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:e0f8269229865c556fd4e7ef029d8c0ae9cac49fbf26784679a579060fd9a1a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-cli-tools
  tools:
  - MassBank-cli-tools
  - NumPy
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# precision-recall-metric-computation

## Summary

Computes precision, recall, and F-value for MassBank-style spectral searches by classifying top-hit identifications against a similarity-score threshold (0.6 in the paper), using a weighted cosine correlation with ESI-MS2-optimized exponents (m=0.5 on peak intensity, n=2 on m/z). This quantifies how well unmerged versus merged ESI-MS2 reference data identify compounds.

## When to use

Use when evaluating spectral-search identification performance: you have query and reference peak lists (e.g. QqQ vs QqTOF ESI-MS2 spectra), per-pair similarity scores, and need precision/recall/F at a fixed score cutoff such as 0.6.

## When NOT to use

- The reference spectra are already merged across collision-induced dissociation conditions — the merged search is a separate analysis (paper reports merged data improves precision by 21-23% at score 0.6), so unmerged-baseline metrics would not apply directly.
- The input is EI mass spectra intended for the Stein-Scott regime; the ESI-MS2-optimized exponents m=0.5/n=2 are not the appropriate weights there.

## Inputs

- Unmerged ESI-MS2 query peak lists (QS=[QqQ]: 4205 spectra; QS=[QqTOF]: 4431 spectra) for 856 compounds common to the QqQ and QqTOF subsets
- Unmerged ESI-MS2 reference/target peak lists for the same compounds (e.g. from the 3045 ESI-MS2 records of 679 synthetic drugs, January 2010 MassBank release)
- Similarity-score threshold (0.6) and weighting exponents (m, n) for the weighted cosine correlation

## Outputs

- similarity_search_metrics.csv containing precision, recall, and F-value per search direction (TS=[QqQ] and TS=[QqTOF])
- exponent_comparison.csv containing one row per exponent pair (m=0.5/n=2 vs m=0.6/n=3) with precision, recall, and F-value columns

## How to apply

Score every query-target spectrum pair with the weighted cosine correlation function, applying peak-intensity exponent m and m/z exponent n (MassBank's ESI-MS2-optimized pair is m=0.5/n=2; the EI-derived Stein-Scott baseline is m=0.6/n=3). For each search direction (QS=[QqQ]/TS=[QqTOF] and QS=[QqTOF]/TS=[QqQ]), label a query a true match if its top-scoring target is the same compound and the similarity score is at least 0.6, otherwise a false match. From the true/false counts compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F = 2*P*R/(P+R). For the paper's unmerged cross-search of 856 common compounds (4205 spectra QS=[QqQ], 4431 spectra QS=[QqTOF]), the expected metrics are precision 0.222 / recall 0.327 / F 0.264 for TS=[QqQ] and precision 0.276 / recall 0.292 / F 0.284 for TS=[QqTOF]. Write the per-direction (and, if comparing exponents, per-pair) metrics to a CSV such as similarity_search_metrics.csv or exponent_comparison.csv and validate against reported values.

## Related tools

- **pandas** (Loads the MassBank ESI-MS2 query and target peak lists and assembles the metric comparison tables (similarity_search_metrics.csv, exponent_comparison.csv))
- **NumPy** (Implements the weighted cosine correlation similarity score applying peak-intensity exponent m and m/z exponent n as spectral weights)
- **MassBank-cli-tools** (MassBank command-line tools referenced for the spectral search/validation workflow) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
import numpy as np; w = (intensity**0.5) * (mz**2); score = np.dot(w_q, w_t) / (np.linalg.norm(w_q) * np.linalg.norm(w_t)); true_match = (top_hit_compound == compound) & (score >= 0.6)
```

## Evaluation signals

- TS=[QqQ] direction reproduces precision 0.222, recall 0.327, F-value 0.264 at similarity-score threshold 0.6
- TS=[QqTOF] direction reproduces precision 0.276, recall 0.292, F-value 0.284 at threshold 0.6
- exponent_comparison.csv exists with one row per exponent pair and precision/recall/F-value columns, and m=0.5/n=2 precision matches the paper's reported baseline
- Query counts match expectations: 4205 spectra for QS=[QqQ]/TS=[QqTOF] and 4431 for QS=[QqTOF]/TS=[QqQ] over 856 common compounds
- All computed F-values satisfy F = 2*P*R/(P+R) for the corresponding precision and recall

## Limitations

- Unmerged ESI-MS2 data perform poorly as reference spectra (precision 0.222-0.276 at threshold 0.6); merging data under different CID conditions improved identification precision by 21-23%.
- Results are tied to the January 2010 MassBank release (3045 ESI-MS2 records of 679 synthetic drugs) and the QqQ/QqTOF instrument subsets; other releases or instruments will not reproduce the exact metrics.
- The similarity score depends on the weighting-exponent choice; EI-derived Stein-Scott exponents (m=0.6/n=3) yield different metrics than the ESI-MS2-optimized pair (m=0.5/n=2).

## Evidence

- [abstract] the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [results] Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6.: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] 3045 ESI-MS2 data of 679 synthetic drugs from 16 research groups (January 2010): [evidence span withheld — anchor status: not_found]
- [readme] MassBank java command line tools: [evidence span withheld — anchor status: external_doc]
