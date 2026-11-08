---
name: classification-performance-metrics-computation
description: Use when labeled 'relevant' (e.g., natural products, prenylated flavonoids)
  and 'other' (e.g., wheat ear extract) MS/MS reference spectra are available and
  you must quantify how well custom-trained classifiers discriminate the 'other' class.
when_to_use_negative:
- When the training dataset is highly imbalanced — the article warns classification
  of MS/MS spectra into substance classes 'will often not be successful' in that case
  and the package cannot resolve it
- When the goal is de novo annotation or compound identification rather than evaluating
  binary 'relevant'/'other' classifier performance on labeled reference spectra
- When no labeled 'other'-class spectra (e.g., wheat ear extract) are available, since
  specificity cannot be computed without true negatives
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package used to load the labeled MSnLib and BOKU in-house MS/MS spectra
    and to train the LDA, NN, and SVM classifiers with cross-validation and different
    random seeds
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes the structure-informed embeddings of the training spectra that serve
    as classifier input features; embeddings are cached on disk after the first run
- name: uv
  role: Runs the AnnoMe tooling (e.g., classification GUI) in the managed Python environment
    used for the training and metric-computation workflow
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:da1dd6c4a7ec0c5a675225fd29e3e327ca1ac5d3dc855a11b27d0dd75a553ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# classification-performance-metrics-computation

## Summary

Computes cross-validated sensitivity and specificity for binary 'relevant' vs 'other' HR-MS/MS spectrum classifiers (LDA, NN, SVM) trained on MS2DeepScore embeddings, stratified by polarity and collision energy, with majority-vote aggregation across random seeds.

## When to use

Use when labeled 'relevant' (e.g., natural products, prenylated flavonoids) and 'other' (e.g., wheat ear extract) MS/MS reference spectra are available and you must quantify how well custom-trained classifiers discriminate the 'other' class.

## When NOT to use

- When the training dataset is highly imbalanced — the article warns classification of MS/MS spectra into substance classes 'will often not be successful' in that case and the package cannot resolve it
- When the goal is de novo annotation or compound identification rather than evaluating binary 'relevant'/'other' classifier performance on labeled reference spectra
- When no labeled 'other'-class spectra (e.g., wheat ear extract) are available, since specificity cannot be computed without true negatives

## Inputs

- Labeled 'relevant' MS/MS reference spectra (natural products and prenylated flavonoids) from MSnLib libraries
- Labeled 'other' MS/MS reference spectra (BOKU in-house database, wheat ear extract)
- MS2DeepScore structure-informed embeddings of all training spectra
- Per-polarity and per-collision-energy strata of the training data
- Cross-validated classifier outputs (LDA, NN, SVM) across multiple random seeds

## Outputs

- Per-setup (polarity/CE) specificity and sensitivity metrics table saved to cv_metrics.csv
- Majority-vote aggregated classification predictions per MS/MS spectrum

## How to apply

Load the labeled 'relevant' and 'other' reference MS/MS spectra from the MSnLib libraries and the BOKU in-house database, then compute MS2DeepScore structure-informed embeddings for all training spectra, caching them on disk after the first run to avoid the ~4-hour regeneration cost. Stratify the training data separately by polarity and by collision energy, keeping each polarity/CE setup distinct rather than merging CE setups, since classifiers are trained per setup. Train the different classifiers (LDA, NN, SVM) with cross-validation across different random seeds within each polarity/CE stratum, then aggregate the per-seed classifier results by majority vote and compute specificity and sensitivity for each stratified setup. Validate the computed metrics against the reported reference values (specificity minimum 99.4%; sensitivity average 75.7%, minimum 56.4%, maximum 87.7% as in Table ST3) and save the per-setup metrics table to cv_metrics.csv.

## Related tools

- **AnnoMe** (Python package used to load the labeled MSnLib and BOKU in-house MS/MS spectra and to train the LDA, NN, and SVM classifiers with cross-validation and different random seeds) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes the structure-informed embeddings of the training spectra that serve as classifier input features; embeddings are cached on disk after the first run)
- **uv** (Runs the AnnoMe tooling (e.g., classification GUI) in the managed Python environment used for the training and metric-computation workflow) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Cross-validated specificity across all stratified setups reaches at least 99.4%
- Cross-validated sensitivity averages 75.7% with minimum 56.4% and maximum 87.7%, matching Table ST3
- Polarity and collision-energy strata remain distinct — no merged CE setups appear in the per-setup metrics
- cv_metrics.csv contains one specificity/sensitivity row per stratified setup
- MS2DeepScore embeddings are cached on disk after the first run so subsequent runs skip the multi-hour embedding step

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not be successful, especially with highly imbalanced training datasets; the package cannot resolve this
- Sensitivity varies strongly across setups (min 56.4% to max 87.7%), so some polarity/CE classifiers miss a substantial fraction of 'relevant' spectra
- Embedding generation is the most time-consuming step (about 4 hours on a standard laptop) and failed on a less powerful MacBook Air (M4, passive cooling); results should be interpreted with caution when assigning spectra to compound classes

## Evidence

- [results] Cross-validated specificity/sensitivity reference values: [evidence span withheld — anchor status: not_found]
- [methods] Classifier training protocol: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [readme] Majority-vote aggregation of classifier results: "The results are then aggregated and a majority-vote is derived which indicates the final classification of the input MS/MS spectra"
- [readme] Embedding caching rationale: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [abstract] Package purpose: "the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [discussion] Imbalance failure mode: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
