---
name: spectral-dissimilarity-outlier-detection
description: Use when an extract prioritization run includes MS2 spectral data and
  a MEMO dissimilarity matrix, and you need the SC contribution (0–1) to Inventa's
  Priority Score highlighting spectrally dissimilar extracts.
when_to_use_negative:
- No MEMO dissimilarity matrix or MS2 spectral data are available — Inventa then runs
  on the Feature Component (FC) alone and SC cannot be computed.
- The goal is annotation- or literature-based novelty assessment (LC/CC) rather than
  spectral dissimilarity — SC is explicitly annotation-independent and will not reflect
  reported compounds or CANOPUS classes.
- The extract set is not chemodiverse or lacks MS2 fragmentation consistency (e.g.
  spectra acquired under very different conditions without MEMO-comparable fingerprints).
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: MEMO
  role: Generates the MS2-based sample vectorization (peaks and neutral losses) and
    the vectorized dissimilarity matrix that is mined for outliers
  repo: https://github.com/mandelbrot-project/memo
- name: scikit-bio
  role: Dependency used inside the Inventa environment for dissimilarity computations;
    install from conda-forge before creating the environment
- name: Inventa
  role: Runs the machine-learning outlier detection on the MEMO matrix and integrates
    SC into the four-component Priority Score
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:af4c4ef2c53d3169da3f346ac222e938c3829adb50bf0117523d3a827f266098
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - Inventa
  - MEMO
  - MZmine3
  - SIRIUS
  - scikit-bio
  - timaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Spectral dissimilarity outlier detection

## Summary

Computes Inventa's Similarity Component (SC) by mining a MEMO-vectorized MS2 dissimilarity matrix with machine-learning outlier detection algorithms to flag spectrally dissimilar (outlier) extracts within a natural-extract set, contributing one of four components to the Priority Score.

## When to use

Use when an extract prioritization run includes MS2 spectral data and a MEMO dissimilarity matrix, and you need the SC contribution (0–1) to Inventa's Priority Score highlighting spectrally dissimilar extracts.

## When NOT to use

- No MEMO dissimilarity matrix or MS2 spectral data are available — Inventa then runs on the Feature Component (FC) alone and SC cannot be computed.
- The goal is annotation- or literature-based novelty assessment (LC/CC) rather than spectral dissimilarity — SC is explicitly annotation-independent and will not reflect reported compounds or CANOPUS classes.
- The extract set is not chemodiverse or lacks MS2 fragmentation consistency (e.g. spectra acquired under very different conditions without MEMO-comparable fingerprints).

## Inputs

- MEMO vectorized dissimilarity matrix file (e.g. Celastraceae_memomatrix.csv) computed from MS2 spectra (peaks and neutral losses) of the extract set
- Untargeted MS2 metabolomics data from the FBMN/GNPS job underlying the MEMO fingerprints
- Inventa notebook parameters (data_process_origin, ionisation_mode, modulating weights w1–w4)

## Outputs

- Per-extract Similarity Component (SC) value in [0, 1], where 1 marks a spectral outlier
- Outlier/non-outlier classification of each extract within the set
- SC column included in the ranked results .tsv alongside FC, FS, LC, CC, and final Priority Score

## How to apply

Vectorize each sample's MS2 information into an MS2 fingerprint (occurrence of MS2 peaks and neutral losses to the precursor) using the MEMO package, then align fingerprints into a dissimilarity matrix — e.g. Celastraceae_memomatrix.csv for the 76 Celastraceae extracts. Feed the matrix into multiple outlier-detection machine-learning algorithms inside Inventa (src/inventa.py, scikit-bio installed from conda-forge) to flag extracts that are spectrally dissimilar from the set. Score each extract on a 0–1 scale where SC = 1 means the extract is classified as an outlier. Because SC relies only on MS2 peaks and neutral losses without annotations, it is independent of the feature alignment used by the Feature Component and complements FC, LC, and CC. Combine SC with the other components using the user-set modulating weights (default w1 = w2 = w3 = w4 = 1) to obtain the Priority Score, and export per-extract SC values in the results .tsv file.

## Related tools

- **MEMO** (Generates the MS2-based sample vectorization (peaks and neutral losses) and the vectorized dissimilarity matrix that is mined for outliers) — https://github.com/mandelbrot-project/memo
- **scikit-bio** (Dependency used inside the Inventa environment for dissimilarity computations; install from conda-forge before creating the environment)
- **Inventa** (Runs the machine-learning outlier detection on the MEMO matrix and integrates SC into the four-component Priority Score) — https://github.com/luigiquiros/inventa

## Evaluation signals

- SC values fall in [0, 1], with SC = 1 exactly for extracts classified as outliers by the ML algorithms
- SC is independent of the MZmine feature alignment — it must derive solely from the MEMO peaks/neutral-loss matrix, not from the quantitative feature table
- For the Celastraceae case study with default weights (w1 = w2 = w3 = w4 = 1), the top-ranked Pristimera indica roots extract carries SC = 1, matching Table 1 (PS 3.23, FC 0.36, FS 0.37, LC 0.87, CC 1, SC 1)
- The results .tsv contains an SC value for every one of the 76 extracts in the metadata table

## Limitations

- SC depends on a precomputed MEMO dissimilarity matrix, which requires MS2 spectra for the features and a completed FBMN/GNPS job; without it, Inventa degrades to Feature-Component-only ranking.
- MEMO is retention-time agnostic and suits chemodiverse samples with poor feature overlap; results depend on the quality of MS2 peak and neutral-loss counting across the set.
- SC is a complementary score — it does not by itself indicate structural novelty and must be interpreted jointly with FC, LC, and CC.

## Evidence

- [methods] SC definition (methods): "The **Similarity Component (SC)** is a complementary score that compares extracts based on their general MS2 spectral information"
- [readme] MEMO matrix construction (readme): "This metric generates a matrix containing all the MS2 information in the form of peaks and neutral losses without annotations."
- [readme] Outlier detection (readme): "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers)."
- [readme] SC=1 semantics (readme): "An SC value of ‘1’ implies the extract is classified as an outlier within the extract set studied."
- [methods] MEMO tool attribution (methods): "using the [**MEMO**](https://doi.org/10.3389/fbinf.2022.842964) metric"
