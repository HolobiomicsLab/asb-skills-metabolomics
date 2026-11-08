---
name: transformer-inference-similarity-prediction
description: Use when MS/MS query spectra (MGF or h5py) must be scored against a reference
  spectral library with a learned similarity that reflects structural (Tanimoto) similarity,
  e.g. analogue search or molecular networking on GNPS-derived data.
when_to_use_negative:
- When a simple deterministic alignment is sufficient and no trained model is available
  — Cosine or Modified Cosine via matchms-style scoring requires no pre-trained model
- When the goal is de novo structure generation from MS/MS spectra via deep generative
  models — TransExION scores similarity and explains fragments but does not generate
  structures
- When spectra are raw and cannot be preprocessed to the protocol (relative intensity,
  10-1000 Da fragment window) — the model was trained on spectra in this exact format
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: TransExION
  role: Pre-trained PyTorch transformer model whose ms_test_spectral_similarity.py
    script predicts the spectral similarity between query and reference MS/MS spectra
  repo: github.com/banhdzui/TransExION
- name: RDKit
  role: Computes Daylight fingerprint Tanimoto structural similarity used as ground
    truth to validate predicted spectral similarities
- name: h5py
  role: Storage format for large-scale spectra and spectrum pair ID files used at
    inference time
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:1e298f68a5aecfafba1af094b79241e64817066668c7be8d97fd79ba5c17eab4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Matplotlib
  - Modified Cosine similarity
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# transformer-inference-similarity-prediction

## Summary

Runs inference with a pre-trained TransExION transformer model to predict MS/MS spectral similarity scores that correlate with Tanimoto structural similarity, using the attention mechanism to capture mass differences between query and reference product ions rather than only aligned peaks.

## When to use

Use when MS/MS query spectra (MGF or h5py) must be scored against a reference spectral library with a learned similarity that reflects structural (Tanimoto) similarity, e.g. analogue search or molecular networking on GNPS-derived data.

## When NOT to use

- When a simple deterministic alignment is sufficient and no trained model is available — Cosine or Modified Cosine via matchms-style scoring requires no pre-trained model
- When the goal is de novo structure generation from MS/MS spectra via deep generative models — TransExION scores similarity and explains fragments but does not generate structures
- When spectra are raw and cannot be preprocessed to the protocol (relative intensity, 10-1000 Da fragment window) — the model was trained on spectra in this exact format

## Inputs

- Reference (library) MS/MS spectra in h5py format (converted from MGF via ms_convert_mgf2h5py.py)
- Query (testing) MS/MS spectra in h5py format, preprocessed (m/z rounded to 2 decimals, relative intensity %, 0.1% intensity floor, 10-1000 Da range, 3-Da denoising)
- Pre-trained TransExION model file (e.g. GNPS_MassBank.ms.model)
- Optional spectrum pair ID file in h5py format (converted from csv with ms_covert_csv2h5py.py) listing the query-reference pairs to score

## Outputs

- CSV file of predicted spectral similarity scores for each query-reference pair (e.g. GNPS_MassBank_result.csv, similarity_scores.csv)
- When benchmarking: Pearson correlation coefficient r between predicted similarity and Tanimoto ground truth, and predicted-vs-Tanimoto point-cloud plots (Fig. 7 style)

## How to apply

First preprocess spectra per the paper's protocol: round m/z to two decimals, convert intensities to relative abundance (%), retain fragments with 0.1% <= intensity and 10 <= m/z <= 1000 Da, and denoise by keeping only the highest peak within a 3-Da sliding window. Convert MGF files to h5py format with ms_convert_mgf2h5py.py, then call ms_test_spectral_similarity.py with the reference (library) h5py file, the query h5py file, the pre-trained model, and optionally a --pairs file (h5py) restricting which query-reference pairs are scored; if --pairs is empty every query is paired with every reference. The rationale is that TransExION, unlike Cosine, Modified Cosine, Spec2Vec or MS2DeepScore, explicitly encodes the mass difference between query and reference fragments, so it performs best when retrieving structural analogues, especially at high Tanimoto distance cut-offs. For benchmarking, compute the Tanimoto score on Daylight fingerprints with RDKit (ms_compute_structural_similarity.py), apply the binned evaluation protocol (10 equal-width bins between 0.0 and 1.0, at most k=3 pairs per bin per testing spectrum) and report the Pearson correlation r between predicted spectral similarity and the Tanimoto ground truth.

## Related tools

- **TransExION** (Pre-trained PyTorch transformer model whose ms_test_spectral_similarity.py script predicts the spectral similarity between query and reference MS/MS spectra) — github.com/banhdzui/TransExION
- **RDKit** (Computes Daylight fingerprint Tanimoto structural similarity used as ground truth to validate predicted spectral similarities)
- **h5py** (Storage format for large-scale spectra and spectrum pair ID files used at inference time)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- Pearson correlation r between predicted similarity and Tanimoto ground truth on mergedGNPS testing pairs should reach ~0.811, versus ~0.489 (Cosine), ~0.538 (Modified Cosine), ~0.557 (Spec2Vec), ~0.755 (MS2DeepScore)
- Predicted similarities must fall within the [0, 1] range and, when the binned protocol is applied, cover 10 equal-width bins between 0.0 and 1.0 with at most k=3 pairs per bin per testing spectrum (~11,425 mergedGNPS testing pairs)
- Output CSV row count matches the number of requested pairs (all query x reference combinations when --pairs is omitted, or exactly the pairs in the h5py pair ID file)
- Post-hoc explanation heatmaps (ms_explain_spectral_similarity.py) should be generated per pair and show query product ion, reference product ion(s) and their mass difference in each cell

## Limitations

- TransExION is supervised — it requires a pre-trained model trained on structural similarity matrices; it cannot be applied without one
- The evaluation on mergedGNPS shows only 'a slight out-performance over the other methods in correlation with the Tanimoto ground truth'; gains are largest for nearly-identical structure matching (high Tanimoto distance cut-offs)
- Input spectra must match the training preprocessing (m/z rounded to 2 decimals, relative abundance %, 10-1000 Da, 3-Da denoising); large-scale inputs must be converted to h5py format first
- Quantitative metrics and confidence intervals supporting comparative performance claims are reported only via figures/plots; the Tanimoto cut-off for 'nearly-identical structure matching' is not specified in the discussion

## Evidence

- [readme] TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity.: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [readme] TransExION is based on Transformer architecture and provides a post hoc explanation: "TransExION is based on Transformer architecture and provides a ***post hoc explanation*** for its outcome in order to reveal the relationship between fragments."
- [intro] The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
- [discussion] TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [readme] If the parameter --pairs is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity.: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
