---
name: extract-similarity-scoring
description: Use when a priority-ranked screening of natural extracts requires flagging
  extracts that are spectral outliers within the set, from an untargeted LC-MS/MS
  feature-based molecular networking dataset with a MEMO vectorized matrix available.
when_to_use_negative:
- The input is a raw annotated feature table rather than a MEMO vectorized MS2 fingerprint
  matrix — use the Feature Component (FC) on the feature quantification table instead
- Only a single outlier detection algorithm is desired — the method relies on the
  consensus complementarity of LOF, One-Class SVM, and Isolation Forest
- The analysis goal is chemical class novelty rather than general MS2 spectral dissimilarity
  — use the Class Component (CC) with CANOPUS output
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Hosts the Similarity Component computation and aggregates SC with FC, LC,
    and CC into the final priority rank output as a .tsv file
  repo: https://github.com/luigiquiros/inventa
- name: MEMO
  role: Generates the MS2-based sample vectorization (peaks and neutral losses counts)
    and the dissimilarity matrix used as outlier detection input
  repo: https://github.com/mandelbrot-project/memo
- name: scikit-bio
  role: Dependency installed from conda-forge in the Inventa conda environment; outlier
    detection algorithms (LOF, One-Class SVM, Isolation Forest) run from scikit-learn
    on the vectorized matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:3e7f7036c9d715541621a37818f1138da613389d5b4c292501bdc120c3b0d113
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - MEMO
  - scikit-bio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# extract-similarity-scoring

## Summary

Computes the Similarity Component (SC) of the Inventa priority rank by vectorizing each extract's MS2 spectral information with the MEMO metric and mining the resulting dissimilarity matrix with unsupervised outlier detection algorithms to flag spectrally dissimilar extracts.

## When to use

Use when a priority-ranked screening of natural extracts requires flagging extracts that are spectral outliers within the set, from an untargeted LC-MS/MS feature-based molecular networking dataset with a MEMO vectorized matrix available.

## When NOT to use

- The input is a raw annotated feature table rather than a MEMO vectorized MS2 fingerprint matrix — use the Feature Component (FC) on the feature quantification table instead
- Only a single outlier detection algorithm is desired — the method relies on the consensus complementarity of LOF, One-Class SVM, and Isolation Forest
- The analysis goal is chemical class novelty rather than general MS2 spectral dissimilarity — use the Class Component (CC) with CANOPUS output

## Inputs

- MEMO vectorized dissimilarity matrix (e.g. Celastraceae_memomatrix.csv) encoding MS2 peaks and neutral losses per sample
- Metadata table (GNPS FBMN format with ATTRIBUTE_Species and ATTRIBUTE_Organe headers) for sample identifiers

## Outputs

- Binary SC score per extract (1 = outlier, 0 = inlier)
- Tab-separated results file in the results folder containing SC values and sample identifiers with the final priority rank

## How to apply

Load the MEMO vectorized dissimilarity matrix (vectorized_data_filename, e.g. Celastraceae_memomatrix.csv), which encodes each extract's MS2 information as counts of MS2 peaks and neutral losses to the precursor, without annotations and independent of the feature alignment used for the Feature Component. Run multiple unsupervised outlier detection machine learning algorithms from scikit-learn — LOF, One-Class SVM, and Isolation Forest — on this matrix to identify spectrally dissimilar extracts. Assign SC = 1 to any extract flagged as an outlier by at least one of the three algorithms and SC = 0 to inliers, then contribute this binary component (weight wn modulable by the user) to the final priority rank which is the addition of the four components FC, LC, CC, and SC. Write the SC values together with sample identifiers to a tab-separated file in the results folder, alongside the other component scores and final rank.

## Related tools

- **Inventa** (Hosts the Similarity Component computation and aggregates SC with FC, LC, and CC into the final priority rank output as a .tsv file) — https://github.com/luigiquiros/inventa
- **MEMO** (Generates the MS2-based sample vectorization (peaks and neutral losses counts) and the dissimilarity matrix used as outlier detection input) — https://github.com/mandelbrot-project/memo
- **scikit-bio** (Dependency installed from conda-forge in the Inventa conda environment; outlier detection algorithms (LOF, One-Class SVM, Isolation Forest) run from scikit-learn on the vectorized matrix)

## Examples

```
conda env create -n inventa -f inventa/environment.yml && conda activate inventa; jupyter lab notebook/inventa.ipynb  # set vectorized_data_filename='Celastraceae_memomatrix.csv' and run the SC outlier detection cell
```

## Evaluation signals

- The output .tsv exists, is tab-separated, and contains an SC column aligned with sample identifiers
- Every extract has SC ∈ {0, 1}: exactly 1 if flagged as outlier by at least one of LOF, One-Class SVM, or Isolation Forest, otherwise 0
- In the Celastraceae proof-of-concept, the Pristimera indica roots extract receives SC = 1, consistent with its published priority rank as a promising source of potentially novel compounds
- The SC computation consumed the MEMO matrix file specified in vectorized_data_filename (e.g. Celastraceae_memomatrix.csv), not the MZmine quantification table
- SC is reported independently of the feature alignment used in FC, confirming it derives solely from MS2 spectral fingerprints

## Limitations

- SC is binary (0/1): it only marks outliers and does not quantify a graded degree of spectral dissimilarity
- Requires an external MEMO vectorization step (MEMO package output format) as an optional input; without it only the Feature Component (FC) can be computed
- Outlier detection is unsupervised, so results depend on the extract set composition: an extract may only appear dissimilar relative to the studied set
- Binder instances shut down after 15 min of inactivity, so parameters and results must be saved locally

## Evidence

- [methods] The Similarity Component (SC) is a complementary score that compares extracts based on their general MS2 spectral information: "The **Similarity Component (SC)** is a complementary score that compares extracts based on their general MS2 spectral information"
- [readme] SC outlier detection via MEMO matrix and ML algorithms: "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers). An SC value of ‘1’ implies the extract is classified as an"
- [methods] MEMO metric generates peaks and neutral losses matrix without annotations: "using the [**MEMO**](https://doi.org/10.3389/fbinf.2022.842964) metric"
- [abstract] Pristimera indica highlighted as promising source of novel compounds: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds"
- [readme] scikit-bio install from conda-forge: "If you have an error, try installing `scikit-bio` from `conda-forge` before creating the environment:"
