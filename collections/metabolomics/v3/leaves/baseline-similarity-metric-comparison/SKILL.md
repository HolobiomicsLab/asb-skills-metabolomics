---
name: baseline-similarity-metric-comparison
description: Use when evaluating a new or existing MS/MS spectral similarity metric
  against established baselines (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore)
  on spectra paired with known structures, to test whether predicted spectral similarity
  correlates with structural (Tanimoto) similarity.
when_to_use_negative:
- Input spectra lack associated compound structures (SMILES), so no Tanimoto ground
  truth can be computed for correlation
- The goal is molecular networking or structural-analogue retrieval ranking rather
  than benchmarking similarity metrics against a Tanimoto ground truth
- Spectra have not been preprocessed per the protocol (m/z rounded to 2 decimals,
  relative intensity %, 0.1% <= intensity, 10 <= m/z <= 1000 Da, 3-Da denoising window),
  making baseline scores incomparable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TransExION
  role: 'Method under test: transformer-based spectral similarity with explicit encoding
    of product-ion mass differences; scores query-reference pairs via ms_test_spectral_similarity.py'
  repo: https://github.com/banhdzui/TransExION
- name: Cosine similarity
  role: Heuristic baseline spectral similarity via peak alignment (matchms-style)
- name: Modified Cosine similarity
  role: Heuristic baseline accounting for neutral losses in peak alignment
- name: Spec2Vec
  role: Unsupervised embedding-based baseline spectral similarity
- name: MS2DeepScore
  role: Supervised deep-learning baseline spectral similarity model
- name: RDKit
  role: Computes Daylight fingerprints and Tanimoto structural similarity ground truth
    via ms_compute_structural_similarity.py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:41ea31fd5bf9c034e50459b5a22da0631d13b3dd87e92b33e2ce6a2480f45bad
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# baseline-similarity-metric-comparison

## Summary

Benchmarks a learned MS/MS spectral similarity metric (TransExION) against heuristic baselines (Cosine, Modified Cosine), an unsupervised embedding method (Spec2Vec), and a supervised deep-learning method (MS2DeepScore) using Tanimoto structural similarity on Daylight fingerprints as ground truth, quantified by Pearson correlation over binned testing pairs.

## When to use

Use when evaluating a new or existing MS/MS spectral similarity metric against established baselines (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) on spectra paired with known structures, to test whether predicted spectral similarity correlates with structural (Tanimoto) similarity.

## When NOT to use

- Input spectra lack associated compound structures (SMILES), so no Tanimoto ground truth can be computed for correlation
- The goal is molecular networking or structural-analogue retrieval ranking rather than benchmarking similarity metrics against a Tanimoto ground truth
- Spectra have not been preprocessed per the protocol (m/z rounded to 2 decimals, relative intensity %, 0.1% <= intensity, 10 <= m/z <= 1000 Da, 3-Da denoising window), making baseline scores incomparable

## Inputs

- Testing/query MS/MS spectra in MGF format (e.g., mergedGNPS test split)
- Reference (training) MS/MS spectra in MGF format, converted to h5py via ms_convert_mgf2h5py.py
- SMILES/structures of query and reference compounds (for RDKit Daylight fingerprint Tanimoto computation)
- Pre-trained TransExION model file (e.g., GNPS_MassBank.ms.model)
- Spectrum pair id file (csv convertible to h5py via ms_covert_csv2h5py.py) restricting evaluated pairs

## Outputs

- Predicted spectral similarity scores per query-reference pair for each method (similarity_scores.csv)
- Tanimoto structural similarity ground-truth matrix per pair
- Pearson correlation coefficients r per method (correlation_results.csv), e.g., TransExION r=0.811 vs Cosine 0.489, Modified Cosine 0.538, Spec2Vec 0.557, MS2DeepScore 0.755
- Predicted-vs-Tanimoto scatter plots (Fig. 7-style point clouds)

## How to apply

Pair each testing spectrum with a subset of reference spectra and score all pairs with the method under test and every baseline: Cosine/Modified Cosine via matchms-style peak alignment, unsupervised Spec2Vec, and supervised MS2DeepScore. Compute the Tanimoto ground truth for every query-reference pair from Daylight fingerprints with RDKit (ms_compute_structural_similarity.py). To keep the evaluation tractable, apply equal-width binning of structural similarity into 10 bins between 0.0 and 1.0 and select at most k=3 pairs per bin per testing spectrum (11,425 mergedGNPS testing pairs in the paper). Finally, compute the Pearson correlation coefficient r between each method's predicted similarity and the Tanimoto ground truth over all testing pairs, and inspect the predicted-vs-Tanimoto point cloud for spread. Rationale: a better metric yields higher r and a tighter point cloud, reflecting the structural similarity the metric is intended to proxy.

## Related tools

- **TransExION** (Method under test: transformer-based spectral similarity with explicit encoding of product-ion mass differences; scores query-reference pairs via ms_test_spectral_similarity.py) — https://github.com/banhdzui/TransExION
- **Cosine similarity** (Heuristic baseline spectral similarity via peak alignment (matchms-style))
- **Modified Cosine similarity** (Heuristic baseline accounting for neutral losses in peak alignment)
- **Spec2Vec** (Unsupervised embedding-based baseline spectral similarity)
- **MS2DeepScore** (Supervised deep-learning baseline spectral similarity model)
- **RDKit** (Computes Daylight fingerprints and Tanimoto structural similarity ground truth via ms_compute_structural_similarity.py)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- TransExION Pearson r should reach approximately 0.811 on mergedGNPS testing pairs, versus 0.489 (Cosine), 0.538 (Modified Cosine), 0.557 (Spec2Vec), and 0.755 (MS2DeepScore)
- The binned pair set should contain 11,425 mergedGNPS testing pairs after selecting at most k=3 pairs per 10 equal-width structural-similarity bins between 0.0 and 1.0 per testing spectrum
- The predicted-vs-Tanimoto scatter for the winning method should show a tighter (less widespread) point cloud than all baselines
- similarity_scores.csv must contain a predicted score for every retained query-reference pair, joinable to correlation_results.csv per method
- Tanimoto values must lie in [0, 1] and correspond to Daylight fingerprint comparisons computed by RDKit

## Limitations

- Baselines have no post-hoc explanation; only TransExION provides fragment-association heatmaps, so comparison is limited to correlation and point-cloud spread
- Evaluation depends on the binning protocol (10 equal-width bins, k=3 pairs per bin per testing spectrum); different pair-sampling schemes change the pair set and correlations
- The paper's discussion reports comparative performance claims without quantitative metrics, p-values, or confidence intervals in that section; the r values come from the evaluation workflow
- MS2DeepScore approaches the learned method's correlation, so the margin over a supervised baseline can be small (0.811 vs 0.755)

## Evidence

- [results] TransExION shows a slight out-performance over the other methods in correlation with the Tanimoto ground truth (higher correlation coefficient r, less widespread point cloud): [evidence span withheld — anchor status: not_found]
- [results] explicit encoding of product ion mass differences: [evidence span withheld — anchor status: not_found]
- [discussion] TransExION outperforms simple, heuristic similarity measures: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model"
- [discussion] outperforms the deep-learning model MS2DeepScore in both test sets: "It also outperforms the deep-learning model MS2DeepScore in both test sets, especially when nearly-identical structure matching is desired"
- [readme] convert a MGF format file into h5py format: "Our library supports a function to convert a MGF format file into h5py format. To do the conversion, run the script ms_convert_mgf2h5py.py with parameters."
- [readme] compute spectral similarity between spectra using a trained TransExION model: "To compute spectral similarity between spectra using a trained TransExION model, the script ms_test_spectral_similarity.py can be used with parameters"
