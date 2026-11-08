---
name: spectral-library-matching
description: Use when you need to quantify how useful predicted spectra are for database
  lookup against a non-overlapping reference library, rather than just comparing predicted
  vs. true spectra directly.
when_to_use_negative:
- You only have one library (no non-overlapping query/main split) — the pipeline asserts
  strict non-overlap between the main library and query library
- You want forward prediction metrics (SDP vs. a ground-truth spectrum for the same
  molecule) — use analysis_pipeline.py instead
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: library_match_pipeline.py
  role: Script that runs the library matching protocol and writes the metrics file
- name: rassp-public
  role: Repository containing the library matching pipeline, sample datasets, and
    metrics module
  repo: https://github.com/thejonaslab/rassp-public
- name: FormulaNet
  role: Source of predicted spectra that are matched against the main library to achieve
    98.0% recall (at 10)
  repo: https://github.com/thejonaslab/rassp-public
- name: NEIMS
  role: Prior state-of-the-art method whose 5.7% lookup error rate (at 10) is the
    baseline being compared against
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:ac6f42a56e532e538461c68bc22444fe8a52c0c97ea7b1381bfa0e7e026b40e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - NEIMS
  - SubsetNet
  - library_match_pipeline.py
  - rassp-public
  - rassp-public (library_match_pipeline.py)
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# spectral-library-matching

## Summary

Evaluates the accuracy of predicted EI–MS spectra by matching them against a separate main spectral library and measuring lookup recall/error rates — the same evaluation used to benchmark FormulaNet (98.0% recall at 10) versus NEIMS (5.7% error rate) on the NIST 2017 Mass Spectral Library.

## When to use

Use when you need to quantify how useful predicted spectra are for database lookup against a non-overlapping reference library, rather than just comparing predicted vs. true spectra directly.

## When NOT to use

- You only have one library (no non-overlapping query/main split) — the pipeline asserts strict non-overlap between the main library and query library
- You want forward prediction metrics (SDP vs. a ground-truth spectrum for the same molecule) — use analysis_pipeline.py instead

## Inputs

- Parquet dataset with columns mol_id, inchi, inchi_key, smiles, rdmol, cv_id, morgan4_crc32, and spect (List[Tuple[Float, Float]])
- Two non-overlapping molecule splits, e.g. smallmols_cfm_pred_public_sample.0.parquet (query) and .1.parquet (main library)
- Predicted spectra file from forward_evaluate_pipeline.py, e.g. forward.preds/<exp>.spect.sqlite

## Outputs

- Metrics file library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> containing error rate at 1 and error rate / recall at 10

## How to apply

Given two non-overlapping sets of molecules (query library and main library, split by inchi_key), represent every molecule by its predicted or experimental spectrum as a list of (mass, intensity) 2-tuples and compute the weighted Stein dot product between each query spectrum and every candidate main-library spectrum. Rank candidates by decreasing SDP and record the rank of the true (inchi_key) match, then derive lookup error rate at 1 and recall/error rate at 10 from those ranks. Optionally restrict candidates with a max-peak mass filter (±5 Da window around the molecular ion) to mimic real database search conditions. The article reports this protocol reduces lookup error (at 10) from 5.7% (NEIMS) to 2.0%, approaching the 1.2% experimental-replicate noise limit, so a correctly run matching should yield recall values in this regime when using FN/SN-predicted spectra against a NIST-like main library.

## Related tools

- **library_match_pipeline.py** (Script that runs the library matching protocol and writes the metrics file)
- **rassp-public** (Repository containing the library matching pipeline, sample datasets, and metrics module) — https://github.com/thejonaslab/rassp-public
- **FormulaNet** (Source of predicted spectra that are matched against the main library to achieve 98.0% recall (at 10)) — https://github.com/thejonaslab/rassp-public
- **NEIMS** (Prior state-of-the-art method whose 5.7% lookup error rate (at 10) is the baseline being compared against)

## Examples

```
python library_match_pipeline.py
```

## Evaluation signals

- The output file library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> exists after running library_match_pipeline.py
- The main library and query library contain strictly non-overlapping inchi_keys (asserted by the pipeline)
- Lookup error rate (at 10) for FormulaNet-predicted spectra falls near 2.0% against a NIST-derived main library, well below the 5.7% NEIMS baseline and approaching the 1.2% experimental replicate limit
- Recorded ranks are consistent: recall at 10 is monotonically related to the fraction of query molecules whose true match ranks ≤ 10

## Limitations

- The pipeline assumes the presence of the inchi_key column in both the main library and query library Parquet files
- Recall is capped by the ~1.2% experiment-to-experiment noise limit when using experimental replicates; predicted spectra cannot exceed this
- Evaluation on the 100-molecule smallmols sample is a tiny subset and may not reflect performance on the full NIST 2017 Main Library

## Evidence

- [readme] Split this 100 mol dataset into 2x 50 mol datasets saved as sample_data/smallmols_cfm_pred_public_sample.0.parquet and .1.parquet. These two non-overlapping datasets are used in the library matching pipeline later on.: "We split this 100 mol dataset into 2x 50 mol datasets saved as `sample_data/smallmols_cfm_pred_public_sample.0.parquet` and `sample_data/smallmols_cfm_pred_public_sample.1.parquet`."
- [readme] The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this.: "The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this."
- [readme] Output goes into library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>.: "Output goes into `library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>`."
- [readme] The pipeline assumes the existence of the inchi_key column in the main_library and query_library Parquet files.: "this pipeline assumes the existence of the `inchi_key` column in the `main_library` and `query_library` Parquet files."
- [abstract] database lookup recall (within top 10 ranked spectra) of 98.0% when evaluated against the NIST 2017 Mass Spectral Library: "database lookup recall (within top 10 ranked spectra) of 98.0% when evaluated against the NIST 2017 Mass Spectral Library"
- [abstract] improve upon the previous state-of-the-art spectral database lookup error rate by a factor of 2.9×, reducing the lookup error (top 10) from 5.7 to 2.0%: "improve upon the previous state-of-the-art spectral database lookup error rate by a factor of 2.9×, reducing the lookup error (top 10) from 5.7 to 2.0%"
- [readme] We take the first 100 molecules from the smallmols dataset [1]. We then run cfm-predict against them and save their spectra in sample_data/smallmols_cfm_pred_public_sample.parquet.: "We take the first 100 molecules from the `smallmols` dataset [1]. We then run `cfm-predict` against them and save their spectra in `sample_data/smallmols_cfm_pred_public_sample.parquet`."
