---
name: weighted-cosine-similarity-scoring
description: Use when scoring ESI-MS2 query spectra against a reference library (e.g.
  3045 MassBank ESI-MS2 records of 679 synthetic drugs) to classify compound identifications
  at a similarity score threshold such as 0.6, or when benchmarking exponent choices.
when_to_use_negative:
- When the goal is EI-MS library searching with the original Stein-Scott unweighted
  comparison rather than ESI-MS2 exponent sensitivity analysis — the EI-derived m=0.6/n=3
  pair is only the baseline here
- When spectra have already been merged across collision-induced dissociation conditions;
  this skill scores the unmerged search (merging is a separate precision-improvement
  step)
- When the identification decision uses a threshold other than 0.6 without re-validating
  precision/recall at the new threshold
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NumPy
  role: Implements the weighted cosine correlation similarity score with peak-intensity
    exponent m and m/z exponent n as spectral weights
- name: pandas
  role: Loads the unmerged MassBank ESI-MS2 query and reference spectra (3045 records
    of 679 synthetic drugs) and assembles the per-pair metrics table
- name: SciPy
  role: Supports the spectral scoring and metric computation workflow in Python
- name: MassBank
  role: Source of the ESI-MS2 reference records and repository hosting the MassBank
    software and record format documentation
  repo: https://github.com/MassBank/MassBank3
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:787394d28756340eeaa321aa12f4d535733a1a56791c991e112d8070aaa4bba3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank
  - NumPy
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# weighted-cosine-similarity-scoring

## Summary

Computes MassBank-style spectral search similarity scores as a weighted cosine correlation between query and reference spectra, with exponents on peak intensity (m) and m/z (n) tuned to ESI-MS2 data (m=0.5, n=2) versus the EI-derived Stein-Scott baseline (m=0.6, n=3).

## When to use

Use when scoring ESI-MS2 query spectra against a reference library (e.g. 3045 MassBank ESI-MS2 records of 679 synthetic drugs) to classify compound identifications at a similarity score threshold such as 0.6, or when benchmarking exponent choices.

## When NOT to use

- When the goal is EI-MS library searching with the original Stein-Scott unweighted comparison rather than ESI-MS2 exponent sensitivity analysis — the EI-derived m=0.6/n=3 pair is only the baseline here
- When spectra have already been merged across collision-induced dissociation conditions; this skill scores the unmerged search (merging is a separate precision-improvement step)
- When the identification decision uses a threshold other than 0.6 without re-validating precision/recall at the new threshold

## Inputs

- MassBank ESI-MS2 query spectra in MassBank record format (accession ID + peaks), e.g. the January 2010 release of 3045 ESI-MS2 records of 679 synthetic drugs
- MassBank ESI-MS2 reference spectra in the same record format
- Exponent pairs to compare: m=0.5/n=2 (MassBank ESI-optimized) and m=0.6/n=3 (Stein-Scott EI-derived)
- Similarity score threshold for identification (0.6)

## Outputs

- Pairwise similarity scores for all query-reference spectrum pairs per exponent pair
- Precision, recall, and F-value per exponent pair at threshold 0.6 (baseline: precision/recall/F of [0.222, 0.327, 0.264] for TS=[QqQ] and [0.276, 0.292, 0.284] for TS=[QqTOF])
- Comparison table (e.g. exponent_comparison.csv) with one row per exponent pair and precision, recall, F-value columns

## How to apply

Load the unmerged MassBank ESI-MS2 query and reference spectra into pandas and, in NumPy, weight each peak by (intensity^m) * (m/z^n) before computing the cosine correlation between weighted query and reference peak vectors. Score all query-reference pairs twice: once with the MassBank ESI-optimized exponent pair m=0.5/n=2, and once with the Stein-Scott EI-MS-derived pair m=0.6/n=3, to quantify sensitivity of the metrics to exponent choice. Classify identifications at a similarity score threshold of 0.6 (the score at which MassBank reports merging improved precision by 21-23%) and compute precision, recall, and F-value per exponent pair. Assemble the per-pair metrics into a comparison table (e.g. exponent_comparison.csv) with one row per exponent pair. MassBank inputs must follow the MassBank record format, with each file containing its own accession ID and peaks.

## Related tools

- **NumPy** (Implements the weighted cosine correlation similarity score with peak-intensity exponent m and m/z exponent n as spectral weights)
- **pandas** (Loads the unmerged MassBank ESI-MS2 query and reference spectra (3045 records of 679 synthetic drugs) and assembles the per-pair metrics table)
- **SciPy** (Supports the spectral scoring and metric computation workflow in Python)
- **MassBank** (Source of the ESI-MS2 reference records and repository hosting the MassBank software and record format documentation) — https://github.com/MassBank/MassBank3

## Examples

```
w = (I**m) * (mz**n); cos = np.dot(wq, wr)/(np.linalg.norm(wq)*np.linalg.norm(wr))  # m,n = (0.5,2) or (0.6,3); ID if cos > 0.6
```

## Evaluation signals

- exponent_comparison.csv exists and contains one row per exponent pair with precision, recall, and F-value columns
- Precision values at threshold 0.6 are consistent with the paper's reported precision effects at that score (merging improved precision by 21-23% at a similarity score of 0.6)
- Unmerged baseline metrics at threshold 0.6 reproduce [0.222, 0.327, 0.264] for TS=[QqQ] and [0.276, 0.292, 0.284] for TS=[QqTOF]
- Both exponent pairs (m=0.5/n=2 and m=0.6/n=3) were run on the identical set of 3045 unmerged ESI-MS2 query/reference records so metrics are directly comparable
- Cosine scores lie in [0, 1] for every query-reference pair, as expected for a weighted cosine correlation

## Limitations

- The MassBank weighting exponents were optimized specifically for ESI-MS2 data; the Stein-Scott m=0.6/n=3 exponents were derived from EI-MS data, so metric differences conflate instrument-type and exponent effects
- Spectral search quality is sensitive to the similarity score threshold (e.g. 0.6); metrics are not transferable across thresholds without re-validation
- Records must conform to the MassBank record format (each file with its own accession ID and peaks); malformed records will fail ingestion

## Evidence

- [abstract] Spectral search similarity scores are calculated by a weighted cosine correlation with weighting exponents on peak intensity and m/z optimized to ESI-MS2 data.: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] Threshold-0.6 precision improvement from merging grounds the choice of 0.6 as the evaluation threshold.: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [readme] Input records must follow the MassBank record format with accession IDs and peaks.: [evidence span withheld — anchor status: not_found]
