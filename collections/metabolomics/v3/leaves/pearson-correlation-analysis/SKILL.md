---
name: pearson-correlation-analysis
description: Use when evaluating a spectral similarity metric against structural (Tanimoto)
  ground truth on MS/MS library testing pairs — e.g. mergedGNPS — to reproduce or
  compare correlation coefficients across multiple scoring methods.
when_to_use_negative:
- When compound structures/fingerprints are unavailable for the spectrum pairs, so
  no Tanimoto ground truth can be computed to correlate against.
- When the goal is molecular networking or structure elucidation explanation rather
  than quantitative benchmarking of similarity-vs-structure correlation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: TransExION
  role: Computes the predicted spectral similarity being benchmarked against Tanimoto
    ground truth
  repo: github.com/banhdzui/TransExION
- name: RDKit
  role: Computes Tanimoto similarity on Daylight fingerprints as the structural ground
    truth
- name: MS2DeepScore
  role: Supervised deep-learning baseline similarity score for comparison
- name: Spec2Vec
  role: Unsupervised baseline similarity score for comparison
- name: PyTorch
  role: Runtime for the TransExION model computing predicted similarities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:66f2c9cf93d235cb69b574c69de3c7c83af7d10ffaad5ec980d3ef4669a902fc
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# pearson-correlation-analysis

## Summary

Benchmark how well a predicted MS/MS spectral similarity (e.g. TransExION) tracks Tanimoto structural similarity on a fixed set of query-reference spectrum pairs, by computing the Pearson correlation coefficient r against the Tanimoto ground truth and comparing to baseline metrics (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore).

## When to use

Use when evaluating a spectral similarity metric against structural (Tanimoto) ground truth on MS/MS library testing pairs — e.g. mergedGNPS — to reproduce or compare correlation coefficients across multiple scoring methods.

## When NOT to use

- When compound structures/fingerprints are unavailable for the spectrum pairs, so no Tanimoto ground truth can be computed to correlate against.
- When the goal is molecular networking or structure elucidation explanation rather than quantitative benchmarking of similarity-vs-structure correlation.

## Inputs

- mergedGNPS testing (query) and training (reference) MS/MS spectra in MGF, converted to h5py via ms_convert_mgf2h5py.py
- Pre-trained TransExION model file (e.g. GNPS_MassBank.ms.model)
- Spectrum pair id file (csv converted to h5py via ms_covert_csv2h5py.py)
- Compound structures (SMILES) for Daylight fingerprint computation

## Outputs

- similarity_scores.csv (predicted spectral similarity per query-reference pair per method)
- correlation_results.csv (Pearson r per method)
- Predicted-vs-Tanimoto point cloud plots (Fig. 7)

## How to apply

Preprocess all MS/MS spectra per the TransExION protocol: round m/z to two decimals, convert intensities to relative abundance (%), keep fragments with 0.1% <= intensity and 10 <= m/z <= 1000 Da, and denoise by keeping only the highest peak within a 3-Da sliding window. Score every query-reference pair with the method under test (via ms_test_spectral_similarity.py) and, in parallel, with the four baselines: Cosine, Modified Cosine and Neutral Loss spectral alignment (matchms-style), unsupervised Spec2Vec, and the supervised deep-learning MS2DeepScore. Compute Tanimoto similarity on Daylight fingerprints for every pair as ground truth using RDKit via ms_compute_structural_similarity.py. Apply the binned evaluation protocol: equal-width binning into 10 bins between 0.0 and 1.0 on structural similarity, selecting at most k=3 pairs per bin per testing spectrum (11,425 mergedGNPS testing pairs). Finally, fit the Pearson correlation coefficient r between each method's predicted spectral similarity and the Tanimoto ground truth, and plot the predicted-vs-Tanimoto point clouds (Fig. 7).

## Related tools

- **TransExION** (Computes the predicted spectral similarity being benchmarked against Tanimoto ground truth) — github.com/banhdzui/TransExION
- **RDKit** (Computes Tanimoto similarity on Daylight fingerprints as the structural ground truth)
- **MS2DeepScore** (Supervised deep-learning baseline similarity score for comparison)
- **Spec2Vec** (Unsupervised baseline similarity score for comparison)
- **PyTorch** (Runtime for the TransExION model computing predicted similarities)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- Pearson r for TransExION on mergedGNPS reaches 0.811 vs 0.489 (Cosine), 0.538 (Modified Cosine), 0.557 (Spec2Vec), and 0.755 (MS2DeepScore).
- After binning (10 equal-width bins in [0.0, 1.0], max k=3 pairs per bin per testing spectrum), the mergedGNPS testing set yields 11,425 pairs — verify this count.
- All predicted similarities should fall in the [0.0, 1.0] range and be finite for every scored pair.
- TransExION's predicted-vs-Tanimoto point cloud should appear less widespread (tighter around the diagonal) than the baselines' clouds in the Fig. 7 plot.

## Limitations

- Correlation is computed on the binned/subsampled pair set (at most k=3 pairs per bin per testing spectrum), not on all possible query-reference pairs.
- Results are tied to the specific mergedGNPS test set and preprocessing thresholds; the article's discussion reports comparative performance claims without quantitative metrics or confidence intervals.
- A higher Pearson r summarizes agreement but does not capture per-bin behavior, e.g. performance at high Tanimoto (nearly-identical structure matching) where TransExION's advantage over MS2DeepScore is most pronounced.

## Evidence

- [results] TransExION shows a slight out-performance over the other methods in correlation with the Tanimoto ground truth (higher correlation coefficient r, less widespread point cloud): [evidence span withheld — anchor status: not_found]
- [discussion] Outperforms baselines: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [discussion] Outperforms MS2DeepScore: "It also outperforms the deep-learning model MS2DeepScore in both test sets, especially when nearly-identical structure matching is desired"
- [readme] README similarity definition: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [readme] README testing command: "python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db  --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv"
