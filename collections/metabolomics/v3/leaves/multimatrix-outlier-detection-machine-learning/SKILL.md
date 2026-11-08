---
name: multimatrix-outlier-detection-machine-learning
description: 'Use when a set of natural extracts needs prioritization for spectral
  dissimilarity: a MEMO vectorized dissimilarity matrix (e.g. Celastraceae_memomatrix.csv)
  is available and the Similarity Component of Inventa''s priority rank must be computed
  from MS2 spectral fingerprints.'
when_to_use_negative:
- When the goal is to score extract novelty from annotated features — that is the
  Feature Component (FC), which uses the MZmine quantification table and GNPS/ISDB/SIRIUS
  annotations, not the MEMO matrix
- When the goal is chemical-class novelty against literature reports — use the Class
  Component (CC, CANOPUS-based) or Literature Component (LC) instead
- When no MEMO vectorized dissimilarity matrix exists — MEMO is an optional input;
  without it Inventa will run only the Feature Component
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3473
tools:
- name: MEMO
  role: Generates the MS2-based sample vectorization (fingerprint matrix of peaks
    and neutral-loss counts) whose dissimilarity matrix is mined for outliers
  repo: https://github.com/mandelbrot-project/memo
- name: scikit-bio
  role: Dependency installed from conda-forge in the Inventa environment to support
    the vectorized-matrix computation
- name: Inventa
  role: Host pipeline that reads the vectorized_data_filename and folds the resulting
    SC into the four-component priority rank
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:f3f73fd76a5a53f6e1af7f09392f5dd64a3eef1ad9352dede570e5cc45212b33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - Inventa
  - MEMO
  - scikit-bio
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multimatrix-outlier-detection-machine-learning

## Summary

Computes Inventa's Similarity Component (SC) by mining a MEMO vectorized MS2 dissimilarity matrix with multiple unsupervised outlier-detection machine-learning algorithms (LOF, One-Class SVM, Isolation Forest) to flag spectrally dissimilar extracts as outliers (SC = 1).

## When to use

Use when a set of natural extracts needs prioritization for spectral dissimilarity: a MEMO vectorized dissimilarity matrix (e.g. Celastraceae_memomatrix.csv) is available and the Similarity Component of Inventa's priority rank must be computed from MS2 spectral fingerprints.

## When NOT to use

- When the goal is to score extract novelty from annotated features — that is the Feature Component (FC), which uses the MZmine quantification table and GNPS/ISDB/SIRIUS annotations, not the MEMO matrix
- When the goal is chemical-class novelty against literature reports — use the Class Component (CC, CANOPUS-based) or Literature Component (LC) instead
- When no MEMO vectorized dissimilarity matrix exists — MEMO is an optional input; without it Inventa will run only the Feature Component

## Inputs

- MEMO vectorized dissimilarity matrix file (vectorized_data_filename, e.g. Celastraceae_memomatrix.csv) — peaks and neutral-loss counts per extract, no annotations
- Metadata table with sample identifiers (GNPS FBMN format, ATTRIBUTE_Species / ATTRIBUTE_Organe headers)

## Outputs

- Binary SC value (1 = outlier, 0 = inlier) per extract
- Tab-separated .tsv results file in the results folder containing the SC column alongside sample identifiers

## How to apply

Load the MEMO vectorized dissimilarity matrix (vectorized_data_filename), which encodes each extract's MS2 information as counts of MS2 peaks and neutral losses (to the precursor) without any annotations, making it retention-time agnostic and independent of the feature alignment used for the Feature Component. Run multiple unsupervised outlier-detection algorithms from scikit-learn — Local Outlier Factor (LOF), One-Class SVM, and Isolation Forest — over the matrix to detect spectrally dissimilar extracts. Assign SC = 1 to extracts classified as outliers by these algorithms and SC = 0 to inliers; note the score is binary, not graded. Write the SC column together with sample identifiers to a tab-separated .tsv file in the results folder. In the Celastraceae proof-of-concept this correctly flags the Pristimera indica (Willd.) A.C.Sm roots extract, which was subsequently shown to yield thirteen new dihydro-β-agarofuran sesquiterpenes.

## Related tools

- **MEMO** (Generates the MS2-based sample vectorization (fingerprint matrix of peaks and neutral-loss counts) whose dissimilarity matrix is mined for outliers) — https://github.com/mandelbrot-project/memo
- **scikit-bio** (Dependency installed from conda-forge in the Inventa environment to support the vectorized-matrix computation)
- **Inventa** (Host pipeline that reads the vectorized_data_filename and folds the resulting SC into the four-component priority rank) — https://github.com/luigiquiros/inventa

## Examples

```
import pandas as pd; from sklearn.ensemble import IsolationForest; X = pd.read_csv('Celastraceae_memomatrix.csv', index_col=0); labels = IsolationForest().fit_predict(X)
```

## Evaluation signals

- The output file exists in the results folder and is tab-separated
- Every SC value is exactly 0 or 1 (binary outlier/inlier assignment), never a fractional score
- The Pristimera indica (Willd.) A.C.Sm roots extract carries SC = 1, matching the published Celastraceae proof-of-concept priority rank
- Sample identifiers in the SC output match the rows/columns of the input MEMO dissimilarity matrix and the metadata table

## Limitations

- SC is a binary 0/1 score: an extract flagged by outlier detection gets 1, everything else 0, so it provides no graded dissimilarity magnitude
- MEMO vectorized data is an optional input; the minimum Inventa workflow (GNPS job + metadata + MZmine quantification table) only computes the Feature Component
- The outlier assignment is unsupervised, so results depend on the behavior of LOF, One-Class SVM, and Isolation Forest on the given extract set rather than on a fixed threshold

## Evidence

- [methods] SC definition in methods: "The **Similarity Component (SC)** is a complementary score that compares extracts based on their general MS2 spectral information"
- [readme] Outlier mining of MEMO matrix: "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers)."
- [readme] SC binary semantics: "An SC value of ‘1’ implies the extract is classified as an outlier within the extract set studied."
- [readme] How the MEMO fingerprint is built: [evidence span withheld — anchor status: external_doc]
