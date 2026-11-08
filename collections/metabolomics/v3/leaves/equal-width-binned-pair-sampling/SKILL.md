---
name: equal-width-binned-pair-sampling
description: Use when a trained spectral-similarity model must be validated against
  Tanimoto structural similarity across the full similarity range on held-out test
  spectrum pairs.
when_to_use_negative:
- Structures are unavailable for the compounds, so Tanimoto scores on Daylight fingerprints
  cannot be computed and no binned ground truth exists.
- The goal is a single aggregate performance number rather than error behaviour as
  a function of structural similarity — per-bin box-plots are unnecessary.
- The pair distribution is intentionally unbalanced (e.g. a library-search retrieval
  setting); forced equal-width sampling with k=3 pairs per bin would distort the operating
  distribution.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: TransExION
  role: Predicts spectral similarity for each sampled spectrum pair via ms_test_spectral_similarity.py
  repo: https://github.com/banhdzui/TransExION
- name: RDKit
  role: Computes Daylight fingerprints and Tanimoto structural-similarity scores used
    for binning and as ground truth
- name: Matplotlib
  role: Draws the per-bin squared-error box-plots with blue MSE markers
- name: MS2DeepScore
  role: Deep-learning baseline similarity measure scored on the same sampled pairs
- name: Spec2Vec
  role: Unsupervised baseline similarity measure scored on the same sampled pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:80b613aa870852fbaafc3a1890f0bc9f7cb330b6d9d1b9317bf9be43a2af6314
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - MS2DeepScore
  - Matplotlib
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - pyteomics
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# equal-width-binned-pair-sampling

## Summary

Evaluates whether predicted MS/MS spectral similarity tracks structural similarity by binning test spectrum pairs into 10 equal-width Tanimoto bins (0.0–1.0, up to k=3 pairs per bin for balance), then plotting per-bin squared-error box-plots with mean-squared-error markers for TransExION and baseline measures.

## When to use

Use when a trained spectral-similarity model must be validated against Tanimoto structural similarity across the full similarity range on held-out test spectrum pairs.

## When NOT to use

- Structures are unavailable for the compounds, so Tanimoto scores on Daylight fingerprints cannot be computed and no binned ground truth exists.
- The goal is a single aggregate performance number rather than error behaviour as a function of structural similarity — per-bin box-plots are unnecessary.
- The pair distribution is intentionally unbalanced (e.g. a library-search retrieval setting); forced equal-width sampling with k=3 pairs per bin would distort the operating distribution.

## Inputs

- mergedGNPS test and reference MS/MS spectra (GNPS-derived, 11,000 unique-compound spectra, preprocessed MGF/h5py)
- Structural (SMILES/InChI) identifiers for test and reference compounds to compute Daylight-fingerprint Tanimoto scores
- Predicted spectral similarities from TransExION, Cosine, Modified Cosine, Spec2Vec and MS2DeepScore for the sampled pairs

## Outputs

- Balanced set of 11,425 test spectrum pairs with Tanimoto bin assignments
- Per-pair squared error (SE) values for each of the five similarity measures
- Per-bin mean squared error (MSE) values
- squared_error_boxplots.png — SE box-plots per Tanimoto bin with blue MSE markers for all five methods

## How to apply

Start from the preprocessed test and reference spectra (m/z rounded to two decimals, intensities as relative abundance %, peaks <0.1% intensity or outside 10–1000 Da removed). For each test spectrum, compute Tanimoto similarity to candidate reference spectra using Daylight fingerprints in RDKit, divide scores into B=10 equal-width bins between 0.0 and 1.0, and sample up to k=3 pairs per bin to build a balanced test-pair set (11,425 pairs for mergedGNPS). Predict spectral similarity for every pair with TransExION (via ms_test_spectral_similarity.py) and the baselines Cosine, Modified Cosine, Spec2Vec and MS2DeepScore. For each pair compute SE = (predicted similarity − Tanimoto score)^2, aggregate MSE per bin, and draw box-plots of SE per Tanimoto bin for all five methods with blue MSE markers in Matplotlib. Rationale: balanced binning prevents the pair set from being dominated by low-similarity pairs, exposing model behaviour (error and variance) at each structural-similarity level, especially for nearly-identical structures.

## Related tools

- **TransExION** (Predicts spectral similarity for each sampled spectrum pair via ms_test_spectral_similarity.py) — https://github.com/banhdzui/TransExION
- **RDKit** (Computes Daylight fingerprints and Tanimoto structural-similarity scores used for binning and as ground truth)
- **Matplotlib** (Draws the per-bin squared-error box-plots with blue MSE markers)
- **MS2DeepScore** (Deep-learning baseline similarity measure scored on the same sampled pairs)
- **Spec2Vec** (Unsupervised baseline similarity measure scored on the same sampled pairs)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- The TransExION box-plots show low squared error with minimal variation across all 10 Tanimoto bins, matching the reported correlation with Tanimoto (r = 0.811 on mergedGNPS).
- The number of sampled pairs equals the balanced-pair total (11,425 for mergedGNPS) with at most k=3 pairs per bin and no empty bin.
- SE is computed as the squared distance between the predicted similarity and the Tanimoto score per pair; MSE per bin is finite and non-negative.
- The figure squared_error_boxplots.png visually matches the reported Fig. 6 SE distributions, with five methods per bin and blue MSE markers.
- Baseline methods (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) should exhibit higher error than TransExION, especially at high Tanimoto bins.

## Limitations

- Equal-width Tanimoto bins rely on enough candidate reference spectra per bin; bins with few near-identical structures may be undersampled despite the k=3 cap per bin.
- Tanimoto on Daylight fingerprints is one structural-similarity proxy; conclusions about 'low prediction error and high stability throughout the entire structure similarity range' are specific to this fingerprint/binning scheme.
- The test is restricted to preprocessed spectra (m/z rounded to two decimals, peaks <0.1% intensity or outside 10–1000 Da removed); raw or differently preprocessed spectra are out of scope.

## Evidence

- [methods] Divide the Tanimoto scores into B=10 equal-width bins between 0.0 and 1.0, compute the squared error (SE = squared distance between predicted and Tanimoto score) per pair per method: [evidence span withheld — anchor status: not_found]
- [methods] pairing each test spectrum with reference spectra via equal-width Tanimoto binning, sampling up to k=3 pairs per bin, computing structural similarity as the Tanimoto score on Daylight fingerprints with RDKit: [evidence span withheld — anchor status: not_found]
- [discussion] TransExION has overall low prediction error and high stability throughout the entire structure similarity range.: [evidence span withheld — anchor status: not_found]
- [readme] To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters: "To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters"
