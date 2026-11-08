---
name: spectral-similarity-scoring
description: Use when MS/MS spectra in MGF or h5py format must be compared to a reference
  spectral library to retrieve structural analogues, or when predicted spectral similarity
  should reflect Daylight-fingerprint Tanimoto structural similarity.
when_to_use_negative:
- When the task requires only molecular networking of already-identified chemical
  families without analogue retrieval scoring thresholds
- When spectra cannot meet the preprocessing assumptions (peaks restricted to 10–1000
  Da, intensities >= 0.1% relative abundance), e.g. very low-mass-cutoff or ultra-sparse
  spectra
- When an explainable per-fragment attribution (post-hoc heatmap of query-reference
  ion associations) is the primary deliverable — use the dedicated explanation workflow
  (ms_explain_spectral_similarity.py) instead
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TransExION
  role: Transformer-based supervised model that predicts spectral similarity correlated
    with Tanimoto structural similarity, considering mass differences between query
    and reference fragments
  repo: https://github.com/banhdzui/TransExION
- name: Cosine similarity
  role: Baseline spectral alignment similarity measure for comparison of retrieval
    precision-recall and correlation with Tanimoto scores
- name: Modified Cosine similarity
  role: Baseline spectral alignment similarity measure accounting for neutral losses,
    used for comparison
- name: Spec2Vec
  role: Unsupervised deep-learning baseline spectral similarity measure used for comparison
- name: MS2DeepScore
  role: Supervised deep-learning baseline spectral similarity measure used for comparison
- name: RDKit
  role: Computes Daylight-fingerprint Tanimoto structural similarity scores as ground
    truth via ms_compute_structural_similarity.py
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:18a94ac61009189685575192a2428cf38a72132ddf6b09f311cc1a89cd668c94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Matplotlib
  - Modified Cosine similarity
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - pyteomics
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# spectral-similarity-scoring

## Summary

Score MS/MS spectrum pairs for spectral similarity using the Transformer-based TransExION model (and baselines Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) so that predicted spectral similarity correlates with Tanimoto structural similarity, enabling retrieval of structural analogues from spectral libraries such as GNPS.

## When to use

Use when MS/MS spectra in MGF or h5py format must be compared to a reference spectral library to retrieve structural analogues, or when predicted spectral similarity should reflect Daylight-fingerprint Tanimoto structural similarity.

## When NOT to use

- When the task requires only molecular networking of already-identified chemical families without analogue retrieval scoring thresholds
- When spectra cannot meet the preprocessing assumptions (peaks restricted to 10–1000 Da, intensities >= 0.1% relative abundance), e.g. very low-mass-cutoff or ultra-sparse spectra
- When an explainable per-fragment attribution (post-hoc heatmap of query-reference ion associations) is the primary deliverable — use the dedicated explanation workflow (ms_explain_spectral_similarity.py) instead

## Inputs

- Query MS/MS spectra in MGF format (converted to h5py)
- Reference/library MS/MS spectra in h5py format (e.g. GNPS_MassBank_train.db)
- Pre-trained TransExION model file (e.g. GNPS_MassBank.ms.model from Zenodo record 8175528)
- Optional spectrum pair ID file in h5py format (from csv via ms_covert_csv2h5py.py)

## Outputs

- CSV of predicted spectral similarity scores per query-reference pair (e.g. GNPS_MassBank_result.csv, similarity_scores.csv)
- Tanimoto structural similarity matrix (.ref file from ms_compute_structural_similarity.py)
- Precision-recall curves and squared-error box-plots across Tanimoto bins (PNG figures)

## How to apply

Convert the MGF spectra to h5py format with ms_convert_mgf2h5py.py, then preprocess per the paper's protocol: round m/z to two decimals, convert intensities to relative abundance (%), keep fragments with 0.1% <= intensity and 10 <= m/z <= 1000 Da, and denoise by retaining only the highest peak within a 3-Da sliding window. Run the pre-trained model (e.g. GNPS_MassBank.ms.model from Zenodo record 8175528) via ms_test_spectral_similarity.py over the query-reference pairs (e.g. 11,425 mergedGNPS test pairs built by equal-width Tanimoto binning into 10 bins between 0.0 and 1.0 with at most k=3 pairs per bin per test spectrum). Score the same pairs with baseline measures — Cosine, Modified Cosine, Spec2Vec, MS2DeepScore — for comparison, and compute Tanimoto ground truth on Daylight fingerprints with RDKit via ms_compute_structural_similarity.py. Evaluate by Pearson correlation between predicted similarity and Tanimoto scores, squared-error distributions across Tanimoto bins, and precision-recall curves at Tanimoto cut-offs 0.6–0.9.

## Related tools

- **TransExION** (Transformer-based supervised model that predicts spectral similarity correlated with Tanimoto structural similarity, considering mass differences between query and reference fragments) — https://github.com/banhdzui/TransExION
- **Cosine similarity** (Baseline spectral alignment similarity measure for comparison of retrieval precision-recall and correlation with Tanimoto scores)
- **Modified Cosine similarity** (Baseline spectral alignment similarity measure accounting for neutral losses, used for comparison)
- **Spec2Vec** (Unsupervised deep-learning baseline spectral similarity measure used for comparison)
- **MS2DeepScore** (Supervised deep-learning baseline spectral similarity measure used for comparison)
- **RDKit** (Computes Daylight-fingerprint Tanimoto structural similarity scores as ground truth via ms_compute_structural_similarity.py)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- Pearson correlation r between predicted spectral similarity and Tanimoto ground truth reaches ~0.811 on mergedGNPS versus 0.489 (Cosine), 0.538 (Modified Cosine), 0.557 (Spec2Vec), 0.755 (MS2DeepScore)
- Squared-error box-plots per Tanimoto bin (B=10 equal-width bins, 0.0–1.0) show low error with minimal variation across bins for TransExION, matching reported Fig. 6
- Precision-recall curves at Tanimoto cut-offs 0.6, 0.7, 0.8, 0.9 show TransExION achieving a notably better precision/recall combination than all four baselines, consistent with reported Fig. 5
- Pair count matches the expected balanced evaluation set (11,425 mergedGNPS test pairs from equal-width Tanimoto binning with k=3 pairs per bin)
- Output CSV contains one predicted similarity per spectrum pair; h5py outputs are loadable and --pairs file indices align with query and reference spectra

## Limitations

- Model performance depends on preprocessing assumptions (m/z rounded to two decimals, relative-abundance intensities, 10–1000 Da range, 0.1% intensity floor, 3-Da denoising window); deviations may degrade correlation with Tanimoto scores
- The model was trained on GNPS/MassBank-derived data; transfer to other spectral libraries or instrument types is not quantified in the provided context
- Evaluation uses balanced pair sampling (equal-width Tanimoto bins, at most k=3 pairs per bin per test spectrum), so scores on unbalanced or exhaustive pair sets may behave differently
- Version of the TransExION model, code environment, and dependencies are not reported in the discussion

## Evidence

- [readme] TransExION is a supervised learning method estimating spectral similarity between MS/MS spectra that are strongly correlated to their structural similarity.: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [discussion] TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [intro] The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
- [results] Preprocessing and pairing protocol from task cards: [evidence span withheld — anchor status: not_found]
- [readme] README usage of testing script: "To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters"
