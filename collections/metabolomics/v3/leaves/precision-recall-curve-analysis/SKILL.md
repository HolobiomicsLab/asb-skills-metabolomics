---
name: precision-recall-curve-analysis
description: Use when comparing similarity metrics (e.g. TransExION, Cosine, Modified
  Cosine, Spec2Vec, MS2DeepScore) on query-reference MS/MS spectrum pairs and the
  goal is analogue-retrieval performance, not a single correlation coefficient.
when_to_use_negative:
- When only a global association between predicted spectral similarity and structural
  similarity is needed (e.g. a Pearson correlation) rather than retrieval performance
  at defined analogue cut-offs
- When the comparison is about post-hoc fragment/heat-map explanation quality rather
  than ranking or retrieval of structural analogues
- When the task is training or validating the TransExION model itself, which uses
  ms_train_spectal_similarity.py with structural-similarity .ref matrices instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3473
tools:
- name: TransExION
  role: Transformer-based model producing predicted spectral similarities for the
    query-reference pairs whose retrieval performance is assessed by precision-recall
    curves
  repo: https://github.com/banhdzui/TransExION
- name: Cosine similarity
  role: Baseline heuristic similarity measure compared against TransExION in the precision-recall
    analysis
- name: Modified Cosine similarity
  role: Baseline heuristic similarity measure compared against TransExION in the precision-recall
    analysis
- name: Spec2Vec
  role: Baseline unsupervised model providing similarity scores for the precision-recall
    comparison
- name: MS2DeepScore
  role: Baseline deep-learning similarity model included in the precision-recall comparison,
    especially at high Tanimoto cut-offs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:539dc6e53eadb013c9279c752cd77fc4dea7a6193a844af2877ea96d73f74328
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Modified Cosine similarity
  - Spec2Vec
  - TransExION
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precision-recall-curve-analysis

## Summary

Evaluates MS/MS spectral similarity measures for structural-analogue retrieval by sweeping the spectral similarity threshold and computing precision and recall at fixed Tanimoto cut-offs (0.6–0.9), as done in the TransExION study on mergedGNPS test pairs.

## When to use

Use when comparing similarity metrics (e.g. TransExION, Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) on query-reference MS/MS spectrum pairs and the goal is analogue-retrieval performance, not a single correlation coefficient.

## When NOT to use

- When only a global association between predicted spectral similarity and structural similarity is needed (e.g. a Pearson correlation) rather than retrieval performance at defined analogue cut-offs
- When the comparison is about post-hoc fragment/heat-map explanation quality rather than ranking or retrieval of structural analogues
- When the task is training or validating the TransExION model itself, which uses ms_train_spectal_similarity.py with structural-similarity .ref matrices instead

## Inputs

- mergedGNPS test MS/MS spectra in MGF format (converted to h5py for TransExION)
- Spectrum pair definitions (spectrum pair id file, convertible from CSV to h5py via ms_covert_csv2h5py.py)
- Pre-trained TransExION model (e.g. GNPS_MassBank.ms.model from Zenodo record 8175528)
- Daylight-fingerprint Tanimoto structural similarity scores for each spectrum pair
- Baseline similarity scores from Cosine, Modified Cosine, Spec2Vec and MS2DeepScore on identical pairs

## Outputs

- Precision-recall curve plot per Tanimoto cut-off (e.g. precision_recall_curves.png, five curves per cut-off)
- Per-pair similarity scores table (e.g. similarity_scores.csv)
- Precision and recall values per method at each of the four Tanimoto cut-offs (0.6, 0.7, 0.8, 0.9)

## How to apply

Compute per-pair spectral similarity scores for all methods on the same spectrum pairs — in TransExION this means converting the mergedGNPS test MGF to h5py with ms_convert_mgf2h5py.py, then running ms_test_spectral_similarity.py with the pre-trained GNPS_MassBank.ms.model (Zenodo record 8175528) over the 11,425 query-reference test pairs generated with equal-width Tanimoto binning and utmost k=3 pairs per bin per test spectrum. Define structural similarity with Daylight-fingerprint Tanimoto scores and bin pairs at four cut-offs (0.6, 0.7, 0.8, 0.9). For each method and each cut-off, vary the spectral similarity threshold and compute precision and recall over the threshold sweep. Plot one precision-recall curve per method per cut-off (mirroring the article's Fig. 5 layout) and write all per-pair scores to a CSV. Correct application is judged by TransExION achieving a better precision/recall combination than all baselines at each cut-off, consistent with the reported TransExION–Tanimoto Pearson r = 0.811.

## Related tools

- **TransExION** (Transformer-based model producing predicted spectral similarities for the query-reference pairs whose retrieval performance is assessed by precision-recall curves) — https://github.com/banhdzui/TransExION
- **Cosine similarity** (Baseline heuristic similarity measure compared against TransExION in the precision-recall analysis)
- **Modified Cosine similarity** (Baseline heuristic similarity measure compared against TransExION in the precision-recall analysis)
- **Spec2Vec** (Baseline unsupervised model providing similarity scores for the precision-recall comparison)
- **MS2DeepScore** (Baseline deep-learning similarity model included in the precision-recall comparison, especially at high Tanimoto cut-offs)

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_test.mgf --out GNPS_MassBank_test.db && python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output similarity_scores.csv
```

## Evaluation signals

- TransExION precision-recall curves dominate the four baselines (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) at every Tanimoto cut-off from 0.6 to 0.9, reproducing the Fig. 5 result
- Number of evaluated pairs matches the 11,425 mergedGNPS query-reference test pairs with utmost k=3 pairs per equal-width Tanimoto bin per test spectrum
- Pearson correlation between TransExION-predicted spectral similarity and Tanimoto score is close to the reported r = 0.811
- All per-pair scores written to similarity_scores.csv are present for every pair and every method, enabling threshold sweeps
- Output PNG contains exactly five precision-recall curves per cut-off, one per method, mirroring the article's figure layout

## Limitations

- The article's discussion reports the comparative performance claim without quantitative metrics, p-values, or confidence intervals, so validation relies on the figure-level result
- The exact Tanimoto cut-off denoting 'nearly-identical structure matching' is not specified in the discussion text; the 0.6–0.9 range comes from the reproduced task specification
- Precision-recall evaluation depends on the Tanimoto binning scheme (equal-width bins, k=3 pairs per bin); changing the binning changes the pair set and results
- Requires a pre-trained model and matched reference/structural-similarity files; results are not transferable without re-deriving Daylight-fingerprint Tanimoto scores for the same pairs

## Evidence

- [discussion] TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [discussion] Comparison includes MS2DeepScore, especially at high Tanimoto cut-off: "It also outperforms the deep-learning model MS2DeepScore in both test sets, especially when nearly-identical structure matching is desired"
- [readme] Input conversion to h5py for large-scale spectral datasets: "To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
- [readme] Testing command for spectral similarity used to generate per-pair scores: "python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db  --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv"
