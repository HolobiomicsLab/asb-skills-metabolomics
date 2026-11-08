---
name: structural-analogue-retrieval-evaluation
description: Use when MS/MS spectra must be searched against a spectral library to
  retrieve structural analogues and compared to baseline similarity measures via precision-recall
  curves at Tanimoto cut-offs from 0.6 to 0.9.
when_to_use_negative:
- The task is spectral similarity prediction for training a model or computing post
  hoc fragment-association explanations rather than benchmarking analogue-retrieval
  performance against baselines.
- Structural similarity cannot be computed (no reference structures available to derive
  Daylight-fingerprint Tanimoto scores), since the precision-recall evaluation depends
  on Tanimoto cut-offs.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TransExION
  role: Transformer-based supervised model whose predicted spectral similarities are
    the evaluated retrieval scores
  repo: github.com/banhdzui/TransExION
- name: Cosine similarity
  role: Baseline heuristic similarity measure compared against TransExION in the precision-recall
    analysis
- name: Modified Cosine similarity
  role: Baseline heuristic similarity measure compared against TransExION in the precision-recall
    analysis
- name: Spec2Vec
  role: Unsupervised baseline spectral similarity model compared against TransExION
- name: MS2DeepScore
  role: Deep-learning baseline spectral similarity model compared against TransExION
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:e05408a10935f45dc2162990376211a0278fc5b9360f14b7535408f11e350fad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Modified Cosine similarity
  - Spec2Vec
  - TransExION
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# structural-analogue-retrieval-evaluation

## Summary

Evaluates how well MS/MS spectral similarity measures (TransExION vs. Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) retrieve structural analogues from a spectral library, using precision-recall analysis at Daylight-fingerprint Tanimoto cut-offs (0.6–0.9).

## When to use

Use when MS/MS spectra must be searched against a spectral library to retrieve structural analogues and compared to baseline similarity measures via precision-recall curves at Tanimoto cut-offs from 0.6 to 0.9.

## When NOT to use

- The task is spectral similarity prediction for training a model or computing post hoc fragment-association explanations rather than benchmarking analogue-retrieval performance against baselines.
- Structural similarity cannot be computed (no reference structures available to derive Daylight-fingerprint Tanimoto scores), since the precision-recall evaluation depends on Tanimoto cut-offs.

## Inputs

- mergedGNPS test spectra in MGF format (query and reference)
- Pre-trained TransExION model file (GNPS_MassBank.ms.model from Zenodo record 8175528)
- Spectrum pair ID file (h5py, convertible from CSV via ms_covert_csv2h5py.py)
- Structural similarity (Daylight-fingerprint Tanimoto) scores for query-reference pairs

## Outputs

- Per-pair spectral similarity scores for all five methods (similarity_scores.csv)
- Precision-recall curves per Tanimoto cut-off (precision_recall_curves.png, mirroring Fig. 5)

## How to apply

Convert the test MGF spectra (e.g. mergedGNPS) to h5py format with ms_convert_mgf2h5py.py from the TransExION repository. Load a pre-trained model such as GNPS_MassBank.ms.model (Zenodo record 8175528) and run ms_test_spectral_similarity.py over the ~11,425 mergedGNPS query-reference test pairs — assembled with equal-width Tanimoto binning and utmost k=3 pairs per bin per test spectrum — to obtain predicted spectral similarities. Compute baseline similarities on the identical pairs with Cosine, Modified Cosine, Spec2Vec and MS2DeepScore. Define high structural similarity with four Tanimoto cut-offs (0.6, 0.7, 0.8, 0.9) on Daylight-fingerprint Tanimoto scores, sweep the spectral-similarity threshold to compute precision and recall per method, and plot five precision-recall curves per cut-off, mirroring Fig. 5 of the paper.

## Related tools

- **TransExION** (Transformer-based supervised model whose predicted spectral similarities are the evaluated retrieval scores) — github.com/banhdzui/TransExION
- **Cosine similarity** (Baseline heuristic similarity measure compared against TransExION in the precision-recall analysis)
- **Modified Cosine similarity** (Baseline heuristic similarity measure compared against TransExION in the precision-recall analysis)
- **Spec2Vec** (Unsupervised baseline spectral similarity model compared against TransExION)
- **MS2DeepScore** (Deep-learning baseline spectral similarity model compared against TransExION)

## Examples

```
python ms_test_spectral_similarity.py --db mergedGNPS_test.db --query mergedGNPS_test.db --pairs mergedGNPS_pairs.db --model GNPS_MassBank.ms.model --output mergedGNPS_result.csv
```

## Evaluation signals

- TransExION precision-recall curves should show a notably better precision/recall combination than Cosine, Modified Cosine, Spec2Vec and MS2DeepScore at each Tanimoto cut-off (0.6, 0.7, 0.8, 0.9), consistent with Fig. 5 (TransExION-Tanimoto Pearson r = 0.811).
- similarity_scores.csv must contain per-pair scores for the same 11,425 mergedGNPS query-reference test pairs for all five methods — baselines must be computed on identical pairs for a fair comparison.
- Each curve family should contain exactly five methods (TransExION plus four baselines) per cut-off, matching the layout of Fig. 5.
- The TransExION model file loaded must be the pre-trained GNPS_MassBank.ms.model from Zenodo record 8175528, and spectra must be in h5py format produced by ms_convert_mgf2h5py.py before running the test script.

## Limitations

- No quantitative metrics, statistics, p-values, or confidence intervals are reported in the discussion to support the comparative performance claims against Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore.
- No accession, DOI, or Zenodo/GitHub record is provided in the discussion for the two independent test sets, so obtaining the exact mergedGNPS test split may require the Zenodo record 8175528.
- The specific Tanimoto cut-off value defining 'nearly-identical structure matching' and the model version/dependency details are not specified in the discussion.

## Evidence

- [abstract] The experimental results show that it improves existing spectral similarity measures in searching and interpreting structural analogues as well as in molecular networking.: "The experimental results show that it improves existing spectral similarity measures in searching and interpreting structural analogues as well as in molecular networking."
- [discussion] TransExION outperforms the baseline similarity measures: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [discussion] TransExION outperforms the deep-learning baseline: "It also outperforms the deep-learning model MS2DeepScore in both test sets, especially when nearly-identical structure matching is desired"
- [readme] MGF to h5py conversion step: "To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
- [readme] Spectral similarity test command: "python ms_test_spectral_similarity.py --db [reference spectrum file] --query [query spectrum file] --pairs [spectrum pair id file] --model [model file] --output [result file]"
