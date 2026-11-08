---
name: stein-dot-product-spectrum-ranking
description: Use when comparing predicted EI-MS spectra against a library of candidate
  spectra and you need top-k lookup error rate or recall (e.g. recall-at-10), ranking
  by weighted Stein dot product.
when_to_use_negative:
- The query and main library share overlapping molecules (inchi_key intersection is
  non-empty); the pipeline asserts strict non-overlap and will fail.
- You need a general similarity metric for clustering or alignment rather than ranked
  top-k lookup evaluation of one query against a fixed library.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: library_match_pipeline.py
  role: 'Executes the ranked-lookup protocol: filters by mass window, computes SDP
    against each main-library entry, records true-match ranks, and writes metrics.'
  repo: https://github.com/thejonaslab/rassp-public
- name: metrics.py
  role: Contains the SDP, DP, and other metric functions used to score spectrum matches
    during ranking.
  repo: https://github.com/thejonaslab/rassp-public
- name: const.py
  role: Holds LIBRARY_MATCH_EXPERIMENTS configuration specifying which predicted-spectra
    experiment to evaluate.
  repo: https://github.com/thejonaslab/rassp-public
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:dc8059087c765a09c5c75bb05493306e89dbd0e59f294266ae9a611afca0433f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - NEIMS
  - SubsetNet
  - const.py
  - library_match_pipeline.py
  - metrics.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# stein-dot-product-spectrum-ranking

## Summary

Ranks candidate library spectra against a query by weighted Stein dot product (SDP) to compute database-lookup error/recall-at-k for EI-MS spectral prediction, as implemented in rassp-public's library_match_pipeline.py.

## When to use

Use when comparing predicted EI-MS spectra against a library of candidate spectra and you need top-k lookup error rate or recall (e.g. recall-at-10), ranking by weighted Stein dot product.

## When NOT to use

- The query and main library share overlapping molecules (inchi_key intersection is non-empty); the pipeline asserts strict non-overlap and will fail.
- You need a general similarity metric for clustering or alignment rather than ranked top-k lookup evaluation of one query against a fixed library.

## Inputs

- query library Parquet file with inchi, inchi_key, mol_id, and spect (List[Tuple[Float, Float]]) columns
- main library Parquet file with inchi_key and spect columns
- predicted-spectra output from forward_evaluate_pipeline.py (e.g. forward.preds/demo.spect.sqlite) referenced via ANALYSIS_EXPERIMENTS in const.py

## Outputs

- library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> metrics file containing error rate at 1 and error rate/recall at 10 per query

## How to apply

Load the query library (predicted spectra) and main library (reference spectra) Parquet files, both requiring an inchi_key column; assert that query and main library molecules are strictly non-overlapping. Apply a max-peak mass filter with a ±5 Da window to restrict candidate matches before scoring. For each query spectrum, compute the weighted Stein dot product against every filtered main-library spectrum, rank rows by decreasing SDP, and record the rank of the true match. From those ranks compute error rate at 1 and error rate/recall at 10, then write results to library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>. The SDP weighting emphasizes larger-intensity peaks, matching the article's forward-prediction accuracy metric.

## Related tools

- **library_match_pipeline.py** (Executes the ranked-lookup protocol: filters by mass window, computes SDP against each main-library entry, records true-match ranks, and writes metrics.) — https://github.com/thejonaslab/rassp-public
- **metrics.py** (Contains the SDP, DP, and other metric functions used to score spectrum matches during ranking.) — https://github.com/thejonaslab/rassp-public
- **const.py** (Holds LIBRARY_MATCH_EXPERIMENTS configuration specifying which predicted-spectra experiment to evaluate.) — https://github.com/thejonaslab/rassp-public

## Examples

```
python library_match_pipeline.py
```

## Evaluation signals

- library_match_results.metrics.mass_filter_15.reg_dp file exists after running library_match_pipeline.py.
- The pipeline's assert that query and main library inchi_key sets are strictly non-overlapping passes.
- For the RASSP:FN (FormulaNet) model, top-10 lookup recall should reach ~98.0% (error rate at 10 ≈ 2.0%), consistent with the abstract.
- Error rate at 10 should fall well below the 5.7% baseline of the previous state of the art.
- Ranked error rate should approach the ~1.2% experiment-to-experiment noise limit reported in the article.

## Limitations

- Requires the inchi_key column to be present in both query and main library Parquet files; the pipeline does not tolerate missing identifiers.
- The comparison assumes the query spectrum is a prediction against a library of experimental spectra, so replicate-noise (~1.2%) bounds achievable recall.
- Mass filter is fixed at a max-peak ±5 Da window; other windows would need code changes in the pipeline.
- SDP weighting is specific to the article's forward-prediction accuracy metric and may not suit other scoring schemes without modification.

## Evidence

- [intro] RASSP:FN) predicts probability distributions over possible chemical subformulae and achieves a state-of-the-art forward prediction accuracy of 92.9% Stein dot product6 and a database lookup recall: "achieves a state-of-the-art forward prediction accuracy of 92.9% Stein dot product6 and a database lookup recall"
- [readme] Library matching metrics: output goes into library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> after running library_match_pipeline.py: "Output goes into `library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>`."
- [readme] The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this.: "The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this."
- [readme] Unlike previous pipelines, this pipeline assumes the existence of the inchi_key column in the main_library and query_library Parquet files.: "this pipeline assumes the existence of the `inchi_key` column in the `main_library` and `query_library` Parquet files."
- [readme] We split this 100 mol dataset into 2x 50 mol datasets saved as sample_data/smallmols_cfm_pred_public_sample.0.parquet and .1.parquet; these two non-overlapping datasets are used in the library matching pipeline later on.: "These two non-overlapping datasets are used in the library matching pipeline later on."
