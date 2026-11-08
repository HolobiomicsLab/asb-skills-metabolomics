---
name: unsupervised-spectral-word2vec-embedding-training
description: Use when building a spectral library search or analogue search pipeline
  that needs unsupervised spectral embeddings over full library spectra (including
  unannotated ones), as in MS2Query's Spec2Vec stage on GNPS.
when_to_use_negative:
- You need supervised structural-similarity labels; MS2Deepscore (trained on fully
  annotated GNPS spectra with spectrum pairs labeled by chemical structural similarity)
  is the appropriate model instead
- Your library is small or already has exact-match spectra only — full-library embedding
  training adds cost without analogue-search benefit
- You only need a spectral similarity score for identical molecules; cosine or modified
  cosine score (matchms, 0.05 Da fragment tolerance) suffices
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Spec2Vec
  role: Unsupervised word2vec-style embedding model trained on all library spectra
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: 'Library cleaning: metadata harmonization, peak filtering, intensity normalization,
    and modified cosine score calculations'
- name: MS2Query
  role: Pipeline into which the trained Spec2Vec embeddings are integrated (via the
    ms2query python package)
  repo: https://github.com/iomega/ms2query
- name: GNPS
  role: Source of the public mass spectral library used as training data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:94916b0008f0608e86f8c85007bbafb8a697ccb9e014a6339604568ae7c41241
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - GNPS
  - MS2Query
  - Spec2Vec
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Unsupervised spectral word2vec embedding training (Spec2Vec)

## Summary

Train an unsupervised Spec2Vec word2vec-style embedding model on all GNPS library MS2 spectra so that spectra are embedded in a latent space where spectral similarity proxies chemical similarity, forming the unsupervised similarity component of MS2Query.

## When to use

Use when building a spectral library search or analogue search pipeline that needs unsupervised spectral embeddings over full library spectra (including unannotated ones), as in MS2Query's Spec2Vec stage on GNPS.

## When NOT to use

- You need supervised structural-similarity labels; MS2Deepscore (trained on fully annotated GNPS spectra with spectrum pairs labeled by chemical structural similarity) is the appropriate model instead
- Your library is small or already has exact-match spectra only — full-library embedding training adds cost without analogue-search benefit
- You only need a spectral similarity score for identical molecules; cosine or modified cosine score (matchms, 0.05 Da fragment tolerance) suffices

## Inputs

- GNPS ALL_GNPS.mgf (or ALL_GNPS_NO_PROPOGATED.mgf) library spectra in MGF format
- matchms-cleaned library spectra with uniform metadata and normalized intensities
- InChIKey-based structure-disjoint train/test split manifest

## Outputs

- Trained Spec2Vec model file
- Pre-calculated Spec2Vec embeddings for all library spectra (for fast MS2Deepscore/similarity comparison)
- Training-data-split manifest documenting which spectra were used for training

## How to apply

Clean the GNPS library with matchms (remove spectra with fewer than 3 peaks, remove peaks above 1000 Da and peaks below 0.1% of the highest peak intensity, normalize intensities). Train the unsupervised Spec2Vec model on ALL library spectra — including spectra without structural annotations, since no labels are needed — for 30 epochs with fragment peaks binned to 2 decimal places. Use structure-disjoint train/test splits (InChIKey-based) so that no test spectrum's structure appears in the training set, avoiding data leakage with the supervised models; for benchmarking, retrain Spec2Vec so none of the test spectra were used in training. Save the trained model alongside pre-calculated library embeddings, since pre-computing Spec2Vec embeddings for all library spectra is the main speed optimization of MS2Query.

## Related tools

- **Spec2Vec** (Unsupervised word2vec-style embedding model trained on all library spectra) — https://github.com/iomega/ms2query
- **matchms** (Library cleaning: metadata harmonization, peak filtering, intensity normalization, and modified cosine score calculations)
- **MS2Query** (Pipeline into which the trained Spec2Vec embeddings are integrated (via the ms2query python package)) — https://github.com/iomega/ms2query
- **GNPS** (Source of the public mass spectral library used as training data)

## Examples

```
conda create --name ms2query python=3.9 && conda activate ms2query && pip install ms2query && ms2query --library ./library_folder --download --ionmode positive
```

## Evaluation signals

- Model trained for the documented 30 epochs with fragment peaks binned to 2 decimal places on the full cleaned library
- Train/test split manifest confirms structure-disjointness: no test spectrum's InChIKey structure appears in the Spec2Vec training set
- Pre-calculated embeddings cover all library spectra and enable full-library similarity computation in seconds rather than hours (vs. 9h24m for a 100 Da precursor-difference preselection with modified cosine)
- Downstream MS2Query benchmarking shows analogue/exact-match performance exceeding cosine, modified cosine, and MS2Deepscore alone
- Trained model, library spectra, and split manifest deposited together (Zenodo 10.5281/zenodo.6811540 for raw data and intermediates)

## Limitations

- Unsupervised training uses no structural labels, so its similarity may be a weaker proxy for chemical similarity than the supervised MS2Deepscore on fully annotated spectra
- Hard to objectively judge whether models perform similarly on newly generated data compared to the benchmarking test set
- Library cleaning choices (peaks above 1000 Da removed, <0.1% intensity peaks removed, spectra with <3 peaks removed, >500-peak spectra truncated) constrain what the embedding can learn
- Specific software versions for spec2vec/matchms/rdkit/scikit-learn are not stated in the article

## Evidence

- [methods] Train the unsupervised Spec2Vec model on all library spectra for 30 epochs with fragment peaks binned to 2 decimal places: [evidence span withheld — anchor status: not_found]
- [results] structure-disjoint split rationale: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [discussion] pre-calculated embeddings speed rationale: "The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore."
- [discussion] library cleaning filter steps: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [abstract] embedding-based similarity predictors framing: "a machine learning-based tool that integrates mass spectral embedding-based chemical similarity predictors (Spec2Vec and MS2Deepscore)"
