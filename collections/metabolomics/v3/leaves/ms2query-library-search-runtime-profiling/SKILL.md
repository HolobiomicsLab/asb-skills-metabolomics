---
name: ms2query-library-search-runtime-profiling
description: Use when benchmarking specXplore preprocessing on .mgf LC–MS/MS feature
  lists to measure MS2Query search time, pairwise similarity time (ms2deepscore/modified
  cosine/spec2vec), and output feature counts per dataset.
when_to_use_negative:
- The input spectra are already processed and you only want dashboard-side visualization
  or k-medoid clustering — this skill benchmarks the importing pipeline, not the dashboard
  exploration stage.
- The dataset exceeds roughly 5000 features — the article only reports timing for
  datasets with <1000 and <4000 features after processing; scalability beyond that
  is untested.
- You need defensible ms2deepscore similarity scores on a macOS arm64 machine — the
  reported runtimes are still measurable, but predictions may be unreliable per the
  ms2deepscore issue noted in the README.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Provides the importing pipeline being benchmarked; runs preprocessing on .mgf
    MS/MS feature lists and produces the session data object
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Pretrained-model local library search executed during preprocessing; the dominant
    runtime component (~62 min on the urine dataset)
- name: matchms
  role: Imports spectral data in .mgf format into Python and performs basic specXplore
    data processing before similarity computation
- name: ms2deepscore
  role: Computes pairwise ms2deepscore spectral similarity scores as part of the automatically
    computed pairwise similarities
- name: spec2vec
  role: Computes pairwise spec2vec spectral similarity scores as part of the automatically
    computed pairwise similarities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:9fab859bbec26cbd97470749d1c55a209c0617fc79a93de4311ee19d44e5959f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - spec2vec
  - specXplore
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ms2query-library-search-runtime-profiling

## Summary

Profiles the runtime of specXplore's LC–MS/MS importing pipeline on real feature datasets (wheat plants and urine metabolome .mgf files), benchmarking total preprocessing wall-clock time and its breakdown into MS2Query library search versus pairwise spectral similarity computation, plus the resulting feature counts.

## When to use

Use when benchmarking specXplore preprocessing on .mgf LC–MS/MS feature lists to measure MS2Query search time, pairwise similarity time (ms2deepscore/modified cosine/spec2vec), and output feature counts per dataset.

## When NOT to use

- The input spectra are already processed and you only want dashboard-side visualization or k-medoid clustering — this skill benchmarks the importing pipeline, not the dashboard exploration stage.
- The dataset exceeds roughly 5000 features — the article only reports timing for datasets with <1000 and <4000 features after processing; scalability beyond that is untested.
- You need defensible ms2deepscore similarity scores on a macOS arm64 machine — the reported runtimes are still measurable, but predictions may be unreliable per the ms2deepscore issue noted in the README.

## Inputs

- .mgf LC–MS/MS MS/MS feature file with a feature_id metadata key (e.g. wheat plants and urine metabolome feature lists)
- MS2Query pretrained model and library files for the corresponding ion mode (positive or negative), organized in a 'pos' or 'neg' sub-directory
- Conda environment with specXplore (python=3.8) and its dependencies (matchms, ms2deepscore, ms2query, spec2vec) installed

## Outputs

- Measured total preprocessing runtime per dataset (wall-clock minutes)
- Runtime breakdown: MS2Query library-search time and pairwise spectral similarity computation time
- Post-processing feature count per dataset (e.g. <1000 for wheat, <4000 for urine)
- Benchmark comparison table (e.g. benchmark_table.csv) with one row per dataset: name, measured runtime, measured feature count, and reported constants

## How to apply

Load the wheat and urine .mgf MS/MS feature files into Python with matchms via the specXplore importing pipeline, applying the basic specXplore data processing steps. Execute the full pipeline — the MS2Query pretrained-model library search plus automatic computation of ms2deepscore, modified cosine, and spec2vec pairwise spectral similarity scores — timing the total wall-clock run per dataset on the target hardware. Record the number of features present in each dataset after processing. Compare the measured runtimes and feature counts to the reference ranges from the article: wheat plants <15 min with <1000 features, and urine metabolome <75 min with <4000 features. On a MacBook Pro with Apple M1 Pro (2021), the larger urine dataset split approximately 62 min for MS2Query and 9 min for all pairwise spectral similarity computations, so budget MS2Query as the dominant cost and treat the similarity-matrix step as the secondary term. The specXplore workflow runs in interactive Jupyter notebooks; for reproducibility, note that macOS arm64 systems may yield ms2deepscore similarity predictions not in accordance with results on other systems (windows, ubuntu, macos intel).

## Related tools

- **specXplore** (Provides the importing pipeline being benchmarked; runs preprocessing on .mgf MS/MS feature lists and produces the session data object) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Pretrained-model local library search executed during preprocessing; the dominant runtime component (~62 min on the urine dataset))
- **matchms** (Imports spectral data in .mgf format into Python and performs basic specXplore data processing before similarity computation)
- **ms2deepscore** (Computes pairwise ms2deepscore spectral similarity scores as part of the automatically computed pairwise similarities)
- **spec2vec** (Computes pairwise spec2vec spectral similarity scores as part of the automatically computed pairwise similarities)

## Examples

```
conda activate specxplore_environment && jupyter-notebook notebooks/demo.ipynb
```

## Evaluation signals

- Measured wheat plants runtime is <15 min on the reference MacBook Pro with Apple M1 Pro (2021) hardware.
- Measured urine metabolome runtime is <75 min, with the MS2Query component approximately 62 min and pairwise spectral similarity computations approximately 9 min.
- Post-processing feature count is <1000 for the wheat dataset and <4000 for the urine dataset.
- The pipeline completes without requiring manual similarity computation — specXplore 'automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores.'
- No runtime errors or warnings are emitted; note that on macOS arm64, silent ms2deepscore prediction discrepancies may occur without any error or warning message.

## Limitations

- Reported timings are tied to a specific hardware profile: a MacBook Pro laptop with Apple M1 Pro processor (2021); other systems will vary.
- Benchmarked datasets contained <1000 and <4000 features each after processing; performance for datasets beyond ~5000 features is only anticipated, not tested.
- MS2Query runtime scales with dataset size (~15 min for wheat, ~62 min for urine) and dominates total pipeline time, so profiling on much larger datasets may require substantially more wall-clock time.
- Current specXplore works on macOS and Linux but fails on Windows, so benchmarking there requires the Anaconda PowerShell plus Microsoft Visual C++ Redistributable setup.
- On macOS arm64 computers, the current ms2deepscore package version may produce similarity predictions not in accordance with results on other systems, without any errors or warning messages.

## Evidence

- [results] For the wheat data set, the pipeline preprocessing time was less than 15 min (on a MacBook Pro laptop with Apple M1 Pro processor 2021): "For the wheat data set, the pipeline preprocessing time was less than 15 min (on a MacBook Pro laptop with Apple M1 Pro processor 2021)"
- [results] the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [discussion] to work well for the wheat data and the urine data sets containing <1000 and <4000 features each after processing: "to work well for the wheat data and the urine data sets containing <1000 and <4000 features each after processing"
- [methods] Spectral data are imported into Python using matchms, and basic specXplore data processing is performed: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [methods] automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores.: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] Note that specXplore currently requires a .mgf formatted file with MS/MS spectral data.: [evidence span withheld — anchor status: external_doc]
- [readme] The current ms2deepscore package version may lead to ms2deepscore similarity predictions that are not in accordance with results on other systems (windows, ubuntu, macos intel).: [evidence span withheld — anchor status: external_doc]
- [methods] For the machine learning scores, pretrained models provided with MS2Query are used.: "For the machine learning scores, pretrained models provided with MS2Query are used."
