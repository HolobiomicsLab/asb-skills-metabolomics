---
name: lookup-rank-metrics-computation
description: Use when you have two non-overlapping (by inchi_key) spectral libraries
  — a query library and a main library with (mass, intensity) spectra — and need top-k
  lookup error/recall metrics for predicted versus experimental EI-MS spectra.
when_to_use_negative:
- The query and main library share molecules (overlapping inchi_keys) — the pipeline
  asserts strict non-overlap, and leakage would inflate recall.
- You need forward-prediction quality metrics (Stein dot product between predicted
  and experimental spectra of the same molecule) rather than retrieval metrics — use
  `analysis_pipeline.py`/results.metrics instead.
- Input molecules fall outside the models' supported scope — FormulaNet handles mols
  ≤ 48 atoms and only HCONFSPCl elements, so lookup evaluation of such molecules with
  these pretrained models is out of scope.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rassp-public library_match_pipeline.py
  role: Executes the library matching / database lookup metric pipeline, including
    the mass filter, SDP ranking, and metric file output.
  repo: https://github.com/thejonaslab/rassp-public
- name: rassp const.py
  role: Configuration entry point where LIBRARY_MATCH_EXPERIMENTS (exp_name, main_library,
    query_library, MASS_FILTER_DELTA, DP_NAME) is defined before running the pipeline.
  repo: https://github.com/thejonaslab/rassp-public
- name: FormulaNet (RASSP:FN)
  role: Source of the predicted query spectra whose lookup recall (98.0% at 10) is
    being measured against the main library.
  repo: https://github.com/thejonaslab/rassp-public
- name: SubsetNet (RASSP:SN)
  role: Alternative predicted-spectra model whose lookup recall (95.2% at 10) can
    be substituted into the same pipeline for comparison.
  repo: https://github.com/thejonaslab/rassp-public
- name: NEIMS
  role: Prior state-of-the-art forward model whose lookup performance (5.7% top-10
    error) serves as the baseline the SDP-rank metrics are compared against.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:3779bd74327edb33eb4b4616cfe8831a05c9acdf0aabf37e9c498194bf87c80f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - FormulaNet (RASSP:FN)
  - NEIMS
  - SubsetNet
  - SubsetNet (RASSP:SN)
  - rassp const.py
  - rassp-public library_match_pipeline.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lookup-rank-metrics-computation

## Summary

Computes spectral library-matching (database lookup) metrics by ranking a main library's spectra by decreasing weighted Stein dot product (SDP) against each query spectrum and converting the true match's rank into error rates at 1 and 10. This is the RASSP paper's evaluation of how well predicted EI–MS spectra retrieve the correct compound from a spectral database.

## When to use

Use when you have two non-overlapping (by inchi_key) spectral libraries — a query library and a main library with (mass, intensity) spectra — and need top-k lookup error/recall metrics for predicted versus experimental EI-MS spectra.

## When NOT to use

- The query and main library share molecules (overlapping inchi_keys) — the pipeline asserts strict non-overlap, and leakage would inflate recall.
- You need forward-prediction quality metrics (Stein dot product between predicted and experimental spectra of the same molecule) rather than retrieval metrics — use `analysis_pipeline.py`/results.metrics instead.
- Input molecules fall outside the models' supported scope — FormulaNet handles mols ≤ 48 atoms and only HCONFSPCl elements, so lookup evaluation of such molecules with these pretrained models is out of scope.

## Inputs

- query library Parquet file with columns mol_id, inchi, inchi_key, smiles, rdmol, cv_id, morgan4_crc32, and spect (List[Tuple[Float, Float]] of (mass, intensity) pairs)
- main library Parquet file in the same schema, with the inchi_key column required and molecules strictly non-overlapping with the query library by inchi_key
- predicted spectra from forward inference (forward.preds sqlite with pickled List[Tuple[Float, Float]] spect) referenced via an ANALYSIS_EXPERIMENTS entry in const.py
- MASS_FILTER_DELTA parameter (± Da window around the query's max peak; shipped value 15) and DP_NAME (shipped value reg_dp)

## Outputs

- library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> file containing lookup error rate at 1 and error rate at 10 / recall at 10 for each query spectrum
- per-query ranks of the true inchi_key match within the SDP-sorted main library (intermediate from which the metrics are derived)

## How to apply

Configure `LIBRARY_MATCH_EXPERIMENTS` in `rassp-public/const.py` so `exp_name` points at an existing entry in `ANALYSIS_EXPERIMENTS`, providing `main_library` and `query_library` Parquet files that both contain an `inchi_key` column; the two libraries must be strictly non-overlapping (assert-checked), mirroring the shipped 50-molecule halves `smallmols_cfm_pred_public_sample.0.parquet` and `.1.parquet`. The pipeline applies a max-peak mass filter using a ±MASS_FILTER_DELTA Da window to restrict candidate matches before scoring, then ranks main-library rows for each query by decreasing weighted Stein dot product (SDP) and records the rank of the true inchi_key match. Run `python library_match_pipeline.py`, which writes rank-derived error rate at 1 and error rate at 10 / recall at 10 metrics to `library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>` (defaulted configuration: `mass_filter_15.reg_dp`). Judge success by confirming the output file exists, the mass filter was actually applied (candidate masses within the ±delta window), and the top-10 recall sits in the paper's plausible range — RASSP:FN reaches 98.0% recall at 10 versus NEIMS's 5.7% top-10 error, with 1.2% being the experimental replicate noise limit.

## Related tools

- **rassp-public library_match_pipeline.py** (Executes the library matching / database lookup metric pipeline, including the mass filter, SDP ranking, and metric file output.) — https://github.com/thejonaslab/rassp-public
- **rassp const.py** (Configuration entry point where LIBRARY_MATCH_EXPERIMENTS (exp_name, main_library, query_library, MASS_FILTER_DELTA, DP_NAME) is defined before running the pipeline.) — https://github.com/thejonaslab/rassp-public
- **FormulaNet (RASSP:FN)** (Source of the predicted query spectra whose lookup recall (98.0% at 10) is being measured against the main library.) — https://github.com/thejonaslab/rassp-public
- **SubsetNet (RASSP:SN)** (Alternative predicted-spectra model whose lookup recall (95.2% at 10) can be substituted into the same pipeline for comparison.) — https://github.com/thejonaslab/rassp-public
- **NEIMS** (Prior state-of-the-art forward model whose lookup performance (5.7% top-10 error) serves as the baseline the SDP-rank metrics are compared against.)

## Examples

```
USE_CUDA=0 python rassp/library_match_pipeline.py  # after editing const.py LIBRARY_MATCH_EXPERIMENTS
```

## Evaluation signals

- library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> exists after running `python library_match_pipeline.py`.
- The main library and query library parquet inchi_key sets are disjoint — the pipeline's assert passes without error.
- Every query's candidate set contains only main-library masses within ±MASS_FILTER_DELTA of the query's max peak (mass filter actually applied).
- Computed top-10 recall/error-rate values fall in the paper's plausible band: FN ≈ 98.0% recall at 10, SN ≈ 95.2%, NEIMS baseline ≈ 5.7% error, experimental replicate limit ≈ 1.2% error.
- Metrics are consistent with the recorded true-match ranks: error_rate@k = fraction of queries whose true match rank > k.

## Limitations

- Metrics depend on model generalization: results are only meaningful for molecules within the models' training distribution (HCONFSPCl, ≤48 atoms for FormulaNet).
- The lookup error rate floor is bounded by experiment-to-experiment noise — the paper cites a limiting error rate of 1.2% at rank 10, so metrics cannot go below that.
- Mass filter window (MASS_FILTER_DELTA) and DP_NAME must be chosen carefully; the shipped values (15, reg_dp) are the only ones documented in the README.
- Main/query library parquet files must contain the inchi_key column and be strictly non-overlapping — the pipeline asserts this, so overlapping libraries crash.
- The README marks some output/artifact directories (e.g. Docker options) as TBD, so full reproducibility may rely on local conda install steps and numpy==1.21 pinning.

## Evidence

- [intro] RASSP:FN) predicts probability distributions over possible chemical subformulae and achieves a state-of-the-art forward prediction accuracy of 92.9% Stein dot product6 and a database lookup recall: "RASSP:FN) predicts probability distributions over possible chemical subformulae and achieves a state-of-the-art forward prediction accuracy of 92.9% Stein dot product6 and a database lookup recall"
- [intro] our best model improves the lookup error rate over the prior best forward model by a factor of ∼3×, reducing lookup error rate (at 10) from over 6 to 2%: "our best model improves the lookup error rate over the prior best forward model by a factor of ∼3×, reducing lookup error rate (at 10) from over 6 to 2%, approaching the limiting error rate"
- [intro] RASSP:SN predicts probability distributions over atom subsets and achieves a forward prediction accuracy of 91.8% Stein dot product and a database lookup recall (at 10) of 95.2%: [evidence span withheld — anchor status: not_found]
- [methods] We used the first eight folds for training (2–9, 100 438 molecules, nist-train) and the last 2 folds for validation (0 and 1, 25 205 molecules, nist-test): "We used the first eight folds for training (2–9, 100 438 molecules, nist-train) and the last 2 folds for validation (0 and 1, 25 205 molecules, nist-test)"
- [readme] Edit `const.py` `LIBRARY_MATCH_EXPERIMENTS` with the `exp_name` set to the key for the experiment you want to run library matching metrics on in the `ANALYSIS_EXPERIMENTS`: "Edit `const.py` `LIBRARY_MATCH_EXPERIMENTS` with the `exp_name` set to the key for the experiment you want to run library matching metrics on in the `ANALYSIS_EXPERIMENTS`."
- [readme] The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this.: "The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this."
- [readme] Run and get metrics for the model by running: `python library_match_pipeline.py` Output goes into `library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>`.: "Run and get metrics for the model by running:
`python library_match_pipeline.py`

Output goes into `library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME>`."
- [readme] These two non-overlapping datasets are used in the library matching pipeline later on.: "We split this 100 mol dataset into 2x 50 mol datasets saved as `sample_data/smallmols_cfm_pred_public_sample.0.parquet`
and `sample_data/smallmols_cfm_pred_public_sample.1.parquet`. These two"
