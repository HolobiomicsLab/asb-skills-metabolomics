---
name: lc-msms-feature-count-validation
description: Use when running the specXplore importing pipeline on .mgf MS/MS feature
  lists (e.g. wheat or urine exposome data) and you need a scale sanity check — wall-clock
  preprocessing time and post-processing feature count — before opening the dashboard
  session.
when_to_use_negative:
- The input is raw vendor data that has not yet been converted to .mgf format — convert
  it first with vendor software or MZmine3's GNPS/FBMN/IIN/MN export options before
  any feature-count validation is meaningful.
- Your goal is to reproduce the absolute runtime of the dashboard itself (e.g. t-SNE
  rendering or network redraw under liberal settings) rather than the importing pipeline
  — the paper only anticipates dashboard scalability to ~5000 features and provides
  no dashboard-side benchmarks.
- The dataset comes from an application domain poorly covered by ms2deepscore training
  data — a feature-count check would not catch that the ms2deepscore predictions are
  out of domain.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: specXplore
  role: Provides the importing pipeline whose wall-clock runtime and output feature
    count are the objects being validated
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports .mgf MS/MS spectral data into Python and performs basic specXplore
    data processing during the timed run
- name: MS2Query
  role: Executes the pretrained-model local library search that consumes the majority
    of the reported preprocessing runtime (~62 min of the urine run)
- name: ms2deepscore
  role: Computes the ms2deepscore pairwise similarity scores inside the timed importing
    pipeline; its backend correctness is a known confounder on macos arm64
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:0e7e8e00ac36995a0093c47ee97c5197e1b1d5ad8169f7e2f5cec609f25c0e66
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# lc-msms-feature-count-validation

## Summary

Validates that a specXplore importing-pipeline preprocessing run completed successfully by checking runtime and feature counts against the paper's reported scales: wheat LC–MS/MS <15 min and <1000 features, urine metabolome <75 min and <4000 features. Runtimes are compared by component (MS2Query local library search vs. pairwise similarity computation).

## When to use

Use when running the specXplore importing pipeline on .mgf MS/MS feature lists (e.g. wheat or urine exposome data) and you need a scale sanity check — wall-clock preprocessing time and post-processing feature count — before opening the dashboard session.

## When NOT to use

- The input is raw vendor data that has not yet been converted to .mgf format — convert it first with vendor software or MZmine3's GNPS/FBMN/IIN/MN export options before any feature-count validation is meaningful.
- Your goal is to reproduce the absolute runtime of the dashboard itself (e.g. t-SNE rendering or network redraw under liberal settings) rather than the importing pipeline — the paper only anticipates dashboard scalability to ~5000 features and provides no dashboard-side benchmarks.
- The dataset comes from an application domain poorly covered by ms2deepscore training data — a feature-count check would not catch that the ms2deepscore predictions are out of domain.

## Inputs

- .mgf file containing LC–MS/MS MS/MS spectral data with a 'feature_id' metadata key per feature
- MS2Query pretrained model and library files for positive or negative ion mode (e.g. Zenodo records 10527997 positive, 10528030 negative), placed in mode-specific folders

## Outputs

- Measured total wall-clock preprocessing time for the importing pipeline run, decomposed by component (MS2Query library search time, pairwise spectral similarity computation time)
- Post-processing feature count for the dataset (expected <1000 for wheat, <4000 for urine)
- Generated specXplore session data object saved to disk, ready for the dashboard

## How to apply

Run the specXplore importing pipeline in the conda environment (python 3.8, created via `conda create --name specxplore_environment python=3.8`) on a .mgf MS/MS file, timing the full wall-clock run. The pipeline imports spectra into Python using matchms, performs basic specXplore data processing, then executes the MS2Query pretrained-model library search and automatically computes pairwise spectral similarities using ms2deepscore, modified cosine, and spec2vec. Record the number of features present after processing and attribute elapsed time to the dominant components: on a MacBook Pro with Apple M1 Pro processor (2021), the paper reports the wheat data set (ref. 33,34) took less than 15 min and the larger urine data set (ref. 35) took less than 75 min, of which approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations. Accept the run as scaled correctly if wheat lands under 15 min with <1000 features and urine under 75 min with <4000 features; if runtime or feature count deviates by orders of magnitude, suspect an ms2deepscore backend issue (see README warning on macos arm64 issue 199) or an oversized input.

## Related tools

- **specXplore** (Provides the importing pipeline whose wall-clock runtime and output feature count are the objects being validated) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports .mgf MS/MS spectral data into Python and performs basic specXplore data processing during the timed run)
- **MS2Query** (Executes the pretrained-model local library search that consumes the majority of the reported preprocessing runtime (~62 min of the urine run))
- **ms2deepscore** (Computes the ms2deepscore pairwise similarity scores inside the timed importing pipeline; its backend correctness is a known confounder on macos arm64)

## Examples

```
conda create --name specxplore_environment python=3.8 && conda activate specxplore_environment && pip install "git+https://github.com/kevinmildau/specXplore.git" && time jupyter-notebook notebooks/demo.ipynb
```

## Evaluation signals

- Wheat data set: total preprocessing wall-clock time <15 min AND post-processing feature count <1000 (paper: 'containing <1000 and <4000 features each after processing').
- Urine data set: total preprocessing wall-clock time <75 min AND post-processing feature count <4000.
- Component decomposition sanity check: MS2Query should dominate the runtime — reported as approximately 62 min MS2Query vs. 9 min for all pairwise spectral similarity computations on urine — so a run where similarity computation exceeds MS2Query time suggests a backend problem.
- Hardware reference point: runtimes should be interpreted against the paper's reference system, a MacBook Pro laptop with Apple M1 Pro processor (2021); divergences should be normalized before flagging a failure.
- Session object exists: the importing pipeline must produce a saved specXplore session data object that loads without error into the dashboard; absence of this artifact invalidates the benchmark.

## Limitations

- On macos arm64 systems, ms2deepscore (issue 199) may yield similarity predictions inconsistent with results on windows, ubuntu, or macos intel, silently affecting importing-pipeline runs that rely on default ms2deepscore similarity scores — timing and feature-count checks will not detect this.
- Runtimes are hardware-dependent; the reported benchmarks (wheat <15 min, urine <75 min) were obtained on a single MacBook Pro with Apple M1 Pro processor (2021) and do not generalize to other systems.
- The scope of the feature-count check is limited to datasets of the reported scale (wheat and urine, <1000 and <4000 features respectively); the dashboard itself is only anticipated to scale well to 5000 features and faces difficulties for larger data sets and liberal settings.
- ms2deepscore is only expected to work well for spectra and metabolites well covered by or related to its training data — a passing feature-count check does not validate that the scores are domain-appropriate.

## Evidence

- [results] Wheat runtime and hardware reference: "For the wheat data set, the pipeline preprocessing time was less than 15 min (on a MacBook Pro laptop with Apple M1 Pro processor 2021)"
- [results] Urine runtime decomposition: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [discussion] Feature-count scales: "to work well for the wheat data and the urine data sets containing <1000 and <4000 features each after processing"
- [methods] Pipeline computation contents: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] ms2deepscore arm64 reliability warning: [evidence span withheld — anchor status: external_doc]
- [readme] Environment creation command: [evidence span withheld — anchor status: external_doc]
