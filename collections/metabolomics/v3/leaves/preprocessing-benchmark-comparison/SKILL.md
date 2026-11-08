---
name: preprocessing-benchmark-comparison
description: Use when validating that a specXplore preprocessing run of .mgf MS/MS
  data (e.g. wheat or urine datasets) completes within the published runtime envelope
  and yields the expected number of features after processing.
when_to_use_negative:
- Input is raw LC–MS/MS vendor data rather than a pre-extracted .mgf file with a 'feature_id'
  key — specXplore requires .mgf output from MZmine3 or equivalent preprocessing first.
- The task is visual dashboard exploration (t-SNE, network views, heatmaps) rather
  than measuring importing-pipeline preprocessing time and feature counts.
- Operating on Windows — the README states the current version of specXplore works
  on macOS and Linux but fails on Windows, so benchmark numbers on Windows are not
  comparable to the paper's reported system.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Importing pipeline that processes .mgf data into a specXplore session data
    object, whose runtime and feature count are being benchmarked
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Pretrained-model library search step consuming the majority of preprocessing
    time (~62 min on urine data)
- name: ms2deepscore
  role: Computes pairwise spectral similarities as part of the pipeline's automatic
    similarity score computation
- name: matchms
  role: Imports .mgf spectral data into Python and performs basic specXplore data
    processing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:50fd3e3165d825a34beaa84cae40c3825f5fca8a6ce3257e1a769179b723230f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# Preprocessing benchmark comparison

## Summary

Benchmark the specXplore importing pipeline's preprocessing runtime and feature counts on real LC–MS/MS datasets (wheat and urine .mgf files) by timing the full run — MS2Query library search and pairwise spectral similarity computations — and comparing measured values against the paper's reported ranges (<15 min wheat, <75 min urine).

## When to use

Use when validating that a specXplore preprocessing run of .mgf MS/MS data (e.g. wheat or urine datasets) completes within the published runtime envelope and yields the expected number of features after processing.

## When NOT to use

- Input is raw LC–MS/MS vendor data rather than a pre-extracted .mgf file with a 'feature_id' key — specXplore requires .mgf output from MZmine3 or equivalent preprocessing first.
- The task is visual dashboard exploration (t-SNE, network views, heatmaps) rather than measuring importing-pipeline preprocessing time and feature counts.
- Operating on Windows — the README states the current version of specXplore works on macOS and Linux but fails on Windows, so benchmark numbers on Windows are not comparable to the paper's reported system.

## Inputs

- wheat .mgf MS/MS feature file
- urine .mgf MS/MS feature file (polyphenol exposome study)
- MS2Query pretrained model and library files (positive and negative mode, from Zenodo records 10527997 and 10528030)

## Outputs

- benchmark_table.csv with columns: dataset name, measured preprocessing runtime, measured feature count, reported runtime constants
- per-dataset timing breakdown (MS2Query vs pairwise similarity computation)

## How to apply

Load the wheat and urine .mgf MS/MS feature files into Python with matchms through the specXplore importing pipeline, applying the basic specXplore data processing steps. Execute the full importing pipeline on each dataset — this automatically runs MS2Query pretrained-model library search plus pairwise spectral similarities using ms2deepscore, modified cosine, and spec2vec — and record total wall-clock time per dataset on a MacBook Pro with Apple M1 Pro processor (2021) or equivalent. Record the number of features present in each dataset after processing (wheat <1000, urine <4000). Compare measured runtimes and feature counts to the paper's reported values: wheat <15 min; urine <75 min, of which approximately 62 min is MS2Query and 9 min is all pairwise spectral similarity computations. Write a table with one row per dataset containing dataset name, measured preprocessing runtime, and measured feature count, alongside the reported constants, and save it as benchmark_table.csv.

## Related tools

- **specXplore** (Importing pipeline that processes .mgf data into a specXplore session data object, whose runtime and feature count are being benchmarked) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Pretrained-model library search step consuming the majority of preprocessing time (~62 min on urine data))
- **ms2deepscore** (Computes pairwise spectral similarities as part of the pipeline's automatic similarity score computation)
- **matchms** (Imports .mgf spectral data into Python and performs basic specXplore data processing)

## Examples

```
conda activate specxplore_environment && jupyter-notebook notebooks/demo.ipynb  # run demo.ipynb with wheat/urine .mgf paths; time the pipeline and count features
```

## Evaluation signals

- Measured wheat preprocessing runtime < 15 min on a MacBook Pro (Apple M1 Pro, 2021)
- Measured urine preprocessing runtime < 75 min on the same system
- Urine breakdown: ~62 min attributed to MS2Query and ~9 min to all pairwise spectral similarity computations
- Wheat feature count after processing < 1000; urine feature count after processing < 4000
- benchmark_table.csv contains one row per dataset with measured runtime and feature count alongside the reported constants

## Limitations

- Reference runtimes are hardware-specific (MacBook Pro, Apple M1 Pro, 2021); other systems will differ.
- ms2deepscore on macOS arm64 systems is affected by issue 199 (matchms/ms2deepscore) and may yield unreliable similarity predictions without warning, which directly affects the importing pipeline when using default ms2deepscore scores.
- Scalability beyond ~5000 features is untested; the dashboard is expected to face difficulties for larger datasets, so benchmark results extrapolate poorly to much bigger inputs.

## Evidence

- [results] Wheat runtime below 15 min: "For the wheat data set, the pipeline preprocessing time was less than 15 min (on a MacBook Pro laptop with Apple M1 Pro processor 2021)"
- [results] Urine runtime breakdown: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [discussion] Feature counts after processing: "the wheat data and the urine data sets containing <1000 and <4000 features each after processing"
- [methods] Pipeline prerequisite for dashboard: "Before data can be opened in the specXplore dashboard, it needs to be processed using the specXplore importing pipeline."
- [methods] Pipeline similarity computation scope: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] macOS arm64 ms2deepscore issue: [evidence span withheld — anchor status: external_doc]
- [readme] Windows installation failure: [evidence span withheld — anchor status: external_doc]
