---
name: model-checkpoint-evaluation-via-cli
description: Use when you have predicted spectra (mgf/msp) from a FIORA checkpoint
  and paired reference spectra, and need CLI-driven cosine-similarity metrics per
  test split (NIST/MS-DIAL test split, MSnLib, CASMI 16/22) to benchmark prediction
  quality.
when_to_use_negative:
- You need retention time or CCS accuracy metrics rather than spectral cosine similarity
  — those are separate prediction outputs not covered by the spectral evaluation workflow
  described here
- Spectra were acquired with multi-step (MSn, n>2) fragmentation — FIORA lacks multi-step
  fragmentation support, so its evaluations are limited to single-step MS/MS
- Predictions come from CFM-ID trained on METLIN17 where train/test compound separation
  cannot be guaranteed — interpret such comparisons with the paper's stated caveat
  rather than as a clean generalization test
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FIORA
  role: Provides fiora-predict to generate predicted spectra from the packaged default
    model and fiora-eval to compute the spectral_sqrt_cosine similarity against references
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: Baseline predictor run via the wishartlab/cfmid Docker container; its predictions
    are scored at the closest matching collision energy (10/20/40 eV) for comparison
    in the same evaluation
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: Baseline spectral predictor retrained from the ms-pred repository with test
    compounds excluded from training, evaluated on the same splits
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:0d6d3f57c9ec8ed8dc5c2fefcec87086624b65cfc1fde82302bd17f1e3613dd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Model checkpoint evaluation via CLI

## Summary

Evaluate a trained FIORA spectral-prediction checkpoint against reference spectra through the packaged command-line evaluation tool, computing the square-root-transformed spectral cosine similarity (spectral_sqrt_cosine) and writing per-split result CSVs for downstream aggregation (e.g., Table 1 benchmarks vs. CFM-ID and ICEBERG).

## When to use

Use when you have predicted spectra (mgf/msp) from a FIORA checkpoint and paired reference spectra, and need CLI-driven cosine-similarity metrics per test split (NIST/MS-DIAL test split, MSnLib, CASMI 16/22) to benchmark prediction quality.

## When NOT to use

- You need retention time or CCS accuracy metrics rather than spectral cosine similarity — those are separate prediction outputs not covered by the spectral evaluation workflow described here
- Spectra were acquired with multi-step (MSn, n>2) fragmentation — FIORA lacks multi-step fragmentation support, so its evaluations are limited to single-step MS/MS
- Predictions come from CFM-ID trained on METLIN17 where train/test compound separation cannot be guaranteed — interpret such comparisons with the paper's stated caveat rather than as a clean generalization test

## Inputs

- Predicted spectra in mgf or msp format from fiora-predict (default model under fiora/resources/models)
- Reference MS/MS spectra for the test split (NIST'17+MS-DIAL test split, MSnLib test split, CASMI 2016/2022)
- Query CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type defining the evaluation pairs
- Output directory path (--output-dir) for result CSVs

## Outputs

- Per-split evaluation result CSVs (e.g., test_eval.csv) containing spectral similarity scores
- Median cosine similarity values per model, test set, and ionization mode, aggregatable into a Table 1-like table (e.g., table1_spectral_similarity.csv)

## How to apply

Run fiora-eval after fiora-predict, pointing it at the predicted mgf/msp file, the corresponding reference spectra, and an --output-dir where per-split result CSVs (e.g., test_eval.csv) are written. The default metric is FIORA's spectral_sqrt_cosine, i.e., cosine similarity computed on square-root-transformed intensities and including the precursor peak, which is the metric used to reproduce Table 1. For CASMI 2016, first merge the three stepped collision-energy predictions into a single spectrum; for CASMI 2022, extract challenge spectra with a 10 ppm precursor tolerance and 5-second retention time window and merge replicate measurements at the same collision energy into consensus spectra before scoring. For CFM-ID comparisons, select the prediction at the closest matching collision energy among 10, 20, and 40 eV so all models are scored on comparable inputs. Aggregate median similarity per model, test set, and ionization mode ([M+H]+ / [M-H]-, plus [M]+/[M]- for MSnLib) into a summary table such as table1_spectral_similarity.csv to verify results mirror the paper's benchmark.

## Related tools

- **FIORA** (Provides fiora-predict to generate predicted spectra from the packaged default model and fiora-eval to compute the spectral_sqrt_cosine similarity against references) — https://github.com/BAMeScience/fiora
- **CFM-ID** (Baseline predictor run via the wishartlab/cfmid Docker container; its predictions are scored at the closest matching collision energy (10/20/40 eV) for comparison in the same evaluation) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (Baseline spectral predictor retrained from the ms-pred repository with test compounds excluded from training, evaluated on the same splits) — https://github.com/samgoldman97/ms-pred

## Examples

```
fiora-predict -i queries.csv -o predicted.mgf; fiora-eval -i predicted.mgf -r reference.mgf --output-dir eval_out
```

## Evaluation signals

- Result CSVs (e.g., test_eval.csv) are written to the specified --output-dir for every evaluated split
- Median cosine similarity of FIORA on the test split, MSnLib, and CASMI 16 exceeds the runner-up (CFM-ID or ICEBERG) by roughly 10% to 49%, matching Table 1's reported advantage
- Similarities are computed with the spectral_sqrt_cosine metric: square-root-transformed intensities with the precursor peak included — verify the metric choice before comparing models
- All models are scored on the same collision-energy basis (stepped-CE merge for CASMI 16, closest-CE for CFM-ID, consensus spectra for CASMI 22)
- Aggregated table1_spectral_similarity.csv reproduces the median-similarity structure per model, test set, and ionization mode ([M+H]+, [M-H]-, plus [M]+/[M]- for MSnLib)

## Limitations

- Not all collision energies for all CASMI 2022 challenge spectra were found during extraction, so some challenge evaluations are incomplete
- CFM-ID was trained on the METLIN17 library, so train/test compound separation is not guaranteed for CFM-ID comparisons; retraining CFM-ID with the authors' dataset was computationally infeasible
- NIST'17 is a commercial library requiring a license, limiting public reproduction of the full benchmark
- Retrained baselines must exclude test compounds: all compounds from the test sets were removed from the training data for FIORA and ICEBERG to preserve test integrity

## Evidence

- [results] FIORA's predicted spectra achieve the highest median cosine similarity to reference test spectra on the test split, MSnLib, and CASMI 16 datasets, outperforming the runner-up by 10% to 49% (Table 1).: [evidence span withheld — anchor status: not_found]
- [methods] Evaluation metric: spectral_sqrt_cosine on square-root-transformed intensities including the precursor peak.: [evidence span withheld — anchor status: not_found]
- [discussion] CASMI 16 stepped collision energies merged into a single spectrum before evaluation.: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum."
- [discussion] CASMI 22 challenge spectra extraction parameters: 10 ppm precursor tolerance and 5 second retention time window.: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] CFM-ID predictions evaluated at the closest matching collision energy among 10, 20, and 40 eV.: "Therefore, we evaluated the prediction with the closest matching collision energy, following the approach of Wang et al.23."
- [readme] Default models are packaged under fiora/resources/models and selected automatically by the CLI.: "Default models are packaged under `fiora/resources/models`. The CLI uses these automatically when `--model default` is selected."
- [discussion] Test-set compounds were removed from baseline training data to preserve evaluation integrity.: "To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG."
