---
name: polarity-collision-energy-stratification
description: Use when labeled 'relevant' and 'other' reference HR-MS/MS spectra (e.g.,
  MSnLib libraries, BOKU in-house database, wheat ear extract) span multiple ionization
  polarities and collision energies and must be split into distinct polarity/CE strata
  before classifier training and cross-validation.
when_to_use_negative:
- 'When the training dataset is highly imbalanced: the article and README warn that
  classification of MS/MS spectra into substance classes will most often not be successful
  in that case, and AnnoMe is not capable of resolving this issue'
- When all training spectra share a single polarity and collision energy setup, so
  no stratification is needed
- When the goal is final inference on plant extracts (e.g., Paulownia tomentosa or
  Glycyrrhizza uralensis) rather than validating classifier performance via cross-validation
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: AnnoMe
  role: Python package used to load the labeled 'relevant' and 'other' reference MS/MS
    spectra from MSnLib libraries and the BOKU in-house database, train LDA/NN/SVM
    classifiers with cross-validation and different random seeds, and aggregate results
    by majority vote within each polarity/CE stratum
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings of the training spectra that serve
    as classifier input features; embeddings are computed once and cached on disk
- name: Python
  role: Implementation language of the AnnoMe package used to execute the stratified
    training and cross-validation workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:705115c826dcdf4d86494b231c413e630f26c27f6223dbb4f7d523f8835fee49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# polarity-collision-energy-stratification

## Summary

Stratify labeled HR-MS/MS training spectra by ionization polarity and collision energy so that AnnoMe classifiers (LDA, NN, SVM) are trained and cross-validated within each distinct polarity/CE setup rather than on merged CE setups. This preserves condition-specific spectral behaviour in MS2DeepScore embedding space.

## When to use

Use when labeled 'relevant' and 'other' reference HR-MS/MS spectra (e.g., MSnLib libraries, BOKU in-house database, wheat ear extract) span multiple ionization polarities and collision energies and must be split into distinct polarity/CE strata before classifier training and cross-validation.

## When NOT to use

- When the training dataset is highly imbalanced: the article and README warn that classification of MS/MS spectra into substance classes will most often not be successful in that case, and AnnoMe is not capable of resolving this issue
- When all training spectra share a single polarity and collision energy setup, so no stratification is needed
- When the goal is final inference on plant extracts (e.g., Paulownia tomentosa or Glycyrrhizza uralensis) rather than validating classifier performance via cross-validation

## Inputs

- Labeled 'relevant' and 'other' HR-MS/MS reference spectra from MSnLib spectral libraries (MGF-format spectral data)
- BOKU University in-house MS/MS database spectra
- BOKU University wheat ear extract 'other'-class spectra
- MS2DeepScore structure-informed embeddings for all training spectra (cached on disk after first run)

## Outputs

- Per-polarity and per-collision-energy stratified training splits
- Cross-validated classifier models (LDA, NN, SVM) per polarity/CE stratum across multiple random seeds
- Per-setup specificity and sensitivity metrics table (cv_metrics.csv)

## How to apply

Load the labeled 'relevant' (natural products, prenylated flavonoids) and 'other' (wheat ear extract) MS/MS reference spectra from the MSnLib libraries and the BOKU in-house database with the AnnoMe package, then compute MS2DeepScore structure-informed embeddings for all training spectra, caching them on disk after the first run since this step took ~4 hours on a standard laptop. Stratify the training data separately by polarity and by collision energy, keeping each polarity/CE setup distinct instead of merging CE setups, because spectral fragmentation behaviour differs across these instrument conditions. Within each stratum, train the classifiers (LDA, NN, SVM) using cross-validation across different random seeds, then aggregate per-seed results by majority vote and compute specificity and sensitivity for each stratified setup. Validate against the published benchmarks: cross-validated specificity should reach at least 99.4% and sensitivity should average 75.7% (min 56.4%, max 87.7%) as reported in Table ST3, saving per-setup metrics to cv_metrics.csv.

## Related tools

- **AnnoMe** (Python package used to load the labeled 'relevant' and 'other' reference MS/MS spectra from MSnLib libraries and the BOKU in-house database, train LDA/NN/SVM classifiers with cross-validation and different random seeds, and aggregate results by majority vote within each polarity/CE stratum) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings of the training spectra that serve as classifier input features; embeddings are computed once and cached on disk)
- **Python** (Implementation language of the AnnoMe package used to execute the stratified training and cross-validation workflow)

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Cross-validated specificity per stratified setup reaches at least 99.4%, indicating nearly all 'other'-class spectra are correctly predicted
- Cross-validated sensitivity averages 75.7% across setups, with minimum 56.4% and maximum 87.7% (Table ST3)
- Per-setup metrics table cv_metrics.csv is produced and contains one row per polarity/CE stratum rather than a single merged result
- MS2DeepScore embeddings are cached on disk, so subsequent runs skip the multi-hour embedding-generation step
- Each polarity/CE stratum is trained and cross-validated with multiple random seeds and aggregated by majority vote, not a single-seed split

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and will most often not succeed, especially with highly imbalanced training datasets; AnnoMe cannot resolve this failure mode
- Users must exercise caution with results assigning MS/MS spectra to specific compound classes of interest
- MS2DeepScore embedding generation is the most time-consuming step (took 4 hours on a standard laptop with Intel Core Ultra 5 125U, 12 cores, 16GB RAM; not feasible on a less powerful MacBook Air with passive cooling)

## Evidence

- [methods] different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] The results are then aggregated and a majority-vote is derived: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [abstract] By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra: "By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra"
- [readme] Once these have been calculated, the embeddings are cached on disk: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [supplementary] Cross-validation of the in-house-library-trained classifiers yielded higher specificity (min 99.4%) than sensitivity (average 75.7%, min 56.4%, max 87.7%): [evidence span withheld — anchor status: not_found]
