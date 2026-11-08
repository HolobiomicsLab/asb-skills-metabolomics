---
name: mass-spectral-embedding-quality-estimation
description: Use when computing MS2DeepScore embeddings or cross-ionization-mode Tanimoto
  similarity predictions and you need to flag spectra with low information content
  or dissimilarity to training data before trusting the predicted similarities.
when_to_use_negative:
- Spectra are already annotated/validated and you only need similarity matrices —
  the evaluator adds no information about known-good reference spectra
- You are training a new MS2DeepScore model from scratch rather than assessing predictions
  of an existing one; pair-sampling quality checks apply instead
- The downstream task tolerates unreliable predictions (e.g. exploratory UMAP visualization)
  where filtering would discard informative low-quality spectra
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: Produces the embedding vectors whose quality the embedding evaluator then
    assesses
  repo: https://github.com/matchms/ms2deepscore
- name: MS2DeepScoreEvaluated
  role: Embedding-evaluator wrapper around MS2DeepScore 2.0 that runs the InceptionTime
    quality estimator on each embedding
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the MS2DeepScore and embedding-evaluator models are implemented
    and run
- name: matchms
  role: Provides the spectral filtering pipeline (DEFAULT_FILTERS) that cleans spectra
    before embedding and quality estimation
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:e03aa70a10f0ceb63ba79398b42e2e73d44a28297cad738110e60103e55955a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - MS2DeepScoreEvaluated
  - PyTorch
  - matchms
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectral-embedding-quality-estimation

## Summary

Estimates per-spectrum MS2DeepScore embedding quality by predicting the model's mean squared error for each embedding, so spectra likely to yield unreliable similarity predictions can be flagged or filtered before downstream use.

## When to use

Use when computing MS2DeepScore embeddings or cross-ionization-mode Tanimoto similarity predictions and you need to flag spectra with low information content or dissimilarity to training data before trusting the predicted similarities.

## When NOT to use

- Spectra are already annotated/validated and you only need similarity matrices — the evaluator adds no information about known-good reference spectra
- You are training a new MS2DeepScore model from scratch rather than assessing predictions of an existing one; pair-sampling quality checks apply instead
- The downstream task tolerates unreliable predictions (e.g. exploratory UMAP visualization) where filtering would discard informative low-quality spectra

## Inputs

- Pre-trained MS2DeepScore 2.0 model file (e.g. ms2deepscore_model.pt)
- Cleaned tandem mass spectra (positive and/or negative ionization mode; mgf, msp, mzML or other matchms-supported formats)
- MS2DeepScore embedding vectors (one per spectrum, produced by MS2DeepScore.get_embedding_array)

## Outputs

- Per-spectrum predicted MSE values (embedding-quality scores)
- Flags/labels identifying spectra with unreliable MS2DeepScore predictions
- Filtered similarity matrices or molecular networks after removing high-predicted-MSE spectra

## How to apply

Run the MS2DeepScore 2.0 model to obtain embeddings for each tandem mass spectrum, then feed each embedding through the embedding evaluator — an InceptionTime architecture trained on MS2DeepScoreEvaluated data — to predict the model's MSE for that spectrum. Rank or threshold spectra on the predicted MSE: high values signal low spectral quality, chimeric spectra, limited fragmentation, or spectra that differ substantially from the training data. For downstream analyses such as molecular networking, either remove spectra above a chosen predicted-MSE cutoff or report them as unreliable. As a validation, iteratively removing test spectra with the highest predicted MSE should increase prediction accuracy (lower RMSE between predicted and true Tanimoto similarity) on the remaining spectra, reproducing the paper's Figure 4 result.

## Related tools

- **MS2DeepScore** (Produces the embedding vectors whose quality the embedding evaluator then assesses) — https://github.com/matchms/ms2deepscore
- **MS2DeepScoreEvaluated** (Embedding-evaluator wrapper around MS2DeepScore 2.0 that runs the InceptionTime quality estimator on each embedding) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the MS2DeepScore and embedding-evaluator models are implemented and run)
- **matchms** (Provides the spectral filtering pipeline (DEFAULT_FILTERS) that cleans spectra before embedding and quality estimation) — https://github.com/matchms/ms2deepscore

## Examples

```
model = load_model("ms2deepscore_model.pt"); embeddings = MS2DeepScore(model).get_embedding_array(pipeline.spectra_queries); # then feed embeddings to the MS2DeepScoreEvaluated embedding evaluator to get per-spectrum predicted MSE
```

## Evaluation signals

- Iterative removal of test spectra with the highest predicted MSE must increase MS2DeepScore prediction accuracy (lower RMSE vs true Tanimoto) on the remaining spectra (Figure 4)
- Predicted MSE should correlate with actual MS2DeepScore prediction error on spectra with reference Tanimoto similarities
- Flagged spectra should show the paper's listed failure modes: limited fragmentation, chimeric spectra, or dissimilarity to training data
- Similarity matrix or network computed after filtering should retain high-quality spectra and remove only high-predicted-MSE spectra
- Embedding evaluator runs without error on both positive- and negative-mode embeddings

## Limitations

- Predictions are only as reliable as the evaluator's training (MS2DeepScoreEvaluated) distribution; spectra unlike the training data may still be mis-scored
- Filtering removes spectra and thus reduces network/matrix coverage — trade-off between accuracy and completeness
- Poor predictions are still expected for chimeric mass spectra from multiple precursor ions even after quality filtering
- Risk that the model is not well-suited for very specific tasks or chemical classes

## Evidence

- [intro] Introduce a method to estimate mass spectral embedding quality for each input spectrum so unreliable predictions can be filtered out: "we introduce a method that can estimate the mass spectral embedding quality for each input spectrum"
- [intro] Filter out spectra with high predicted MSE (unreliable embeddings) before using MS2DeepScore predictions: "This allows users to filter out spectra for which the MS2DeepScore predictions are unreliable, e.g., due to low spectral quality or when spectra differ substantially from the training data."
- [abstract] A quality estimation method flags spectra with low information content or dissimilarity to training data: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [discussion] Removing test spectra with high predicted uncertainty to raise overall accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [discussion] Poor predictions expected for low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to training data: "Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions"
- [readme] MS2DeepScore embedding evaluator usage in README: [evidence span withheld — anchor status: external_doc]
