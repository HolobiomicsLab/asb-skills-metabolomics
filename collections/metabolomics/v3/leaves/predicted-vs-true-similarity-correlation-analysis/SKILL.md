---
name: predicted-vs-true-similarity-correlation-analysis
description: Use when a trained MS2DeepScore model must be validated on held-out test
  pairs, especially to check whether cross-ionization-mode Tanimoto predictions correlate
  with ground-truth structural similarity.
when_to_use_negative:
- Spectra that are low-quality, chimeric, or have limited fragmentation — filter these
  with the embedding evaluator first, since poor predictions are expected and will
  depress the correlation
- 'Training-data construction rather than validation: pair sampling for training is
  a separate step (pair_sampling_tutorial) and this skill only scores held-out pairs'
- Datasets restricted to MS1 or lacking annotated structures, since ground-truth Tanimoto
  labels cannot be derived and MS1 data was not included in training
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network producing the predicted Tanimoto scores for test spectrum
    pairs, including across ionization modes
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectral cleaning pipeline (DEFAULT_FILTERS) applied before embedding spectra
    for scoring
  repo: https://github.com/matchms/matchms
- name: PyTorch
  role: Backend in which the MS2DeepScore model is implemented and loaded for prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:fdacf98caedfa836844f4787c8697cecb0585c2d32e9c7468a11d16ea819edf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Predicted vs. true similarity correlation analysis

## Summary

Evaluates a trained MS2DeepScore 2.0 model by predicting Tanimoto similarity for held-out test spectrum pairs — including cross-ionization-mode (positive vs. negative) pairs — and comparing predicted vs. true chemical similarity with Pearson correlation and RMSE, as in Figure 2b of the paper.

## When to use

Use when a trained MS2DeepScore model must be validated on held-out test pairs, especially to check whether cross-ionization-mode Tanimoto predictions correlate with ground-truth structural similarity.

## When NOT to use

- Spectra that are low-quality, chimeric, or have limited fragmentation — filter these with the embedding evaluator first, since poor predictions are expected and will depress the correlation
- Training-data construction rather than validation: pair sampling for training is a separate step (pair_sampling_tutorial) and this skill only scores held-out pairs
- Datasets restricted to MS1 or lacking annotated structures, since ground-truth Tanimoto labels cannot be derived and MS1 data was not included in training

## Inputs

- Held-out test set of tandem MS spectra (mgf/msp/mzml/json/usi) with ionmode and precursor_mz metadata and known structures for ground-truth Tanimoto labels
- Trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt, e.g. from zenodo.org/records/17826815)
- MatchMS-cleaned spectra (DEFAULT_FILTERS pipeline) used as model queries

## Outputs

- Scatter plot of predicted vs. true Tanimoto similarity for cross-ionization-mode test pairs (figure2b_predicted_vs_true_tanimoto.png)
- Correlation metrics table with Pearson r and RMSE (correlation_metrics.csv)
- Numpy similarity matrix of MS2DeepScore predictions between all test spectra

## How to apply

Split spectra into training and held-out test sets before training, keeping known structures so ground-truth Tanimoto similarities can be computed with RDKit from SMILES/InChI. Run the trained model (e.g. ms2deepscore_model.pt from Zenodo) on held-out spectra, supplying ionization mode (as CategoricalToBinary ionmode) and precursor m/z (StandardScaler, mean 0, sd 1000) as additional metadata input, and compute the predicted similarity matrix for both same-mode and cross-mode pairs. Bin pairs by true Tanimoto similarity as done by the balanced pair-sampling algorithm, and separately assess positive-positive, negative-negative, and positive-negative pairs, since the sampling did not enforce balanced ionization-mode pairs. Plot predicted vs. true Tanimoto similarity for cross-ionization-mode pairs (Figure 2b reproduction), compute Pearson r and RMSE, optionally re-computing metrics after removing spectra flagged by the embedding evaluator (high predicted uncertainty/MSE). Save the scatter as figure2b_predicted_vs_true_tanimoto.png and the metrics as correlation_metrics.csv.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network producing the predicted Tanimoto scores for test spectrum pairs, including across ionization modes) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectral cleaning pipeline (DEFAULT_FILTERS) applied before embedding spectra for scoring) — https://github.com/matchms/matchms
- **PyTorch** (Backend in which the MS2DeepScore model is implemented and loaded for prediction)

## Examples

```
from ms2deepscore.models import load_model; from ms2deepscore import MS2DeepScore; model = load_model("ms2deepscore_model.pt"); sims = MS2DeepScore(model).matrix(cleaned_spectra, cleaned_spectra)
```

## Evaluation signals

- Strong correlation between predicted and true Tanimoto similarity on held-out cross-ionization-mode pairs, consistent with Figure 2b
- Pearson r and RMSE recorded in correlation_metrics.csv fall in line with the paper's reported correlation for the same split
- Predicted Tanimoto scores are bounded in [0, 1] and the similarity matrix is symmetric for same-mode evaluations
- Metrics remain strong after filtering spectra with high predicted uncertainty from the embedding evaluator (compare Figure 4 behavior)
- Cross-mode (positive vs. negative) pair performance is assessed separately from negative-negative pairs, which the paper flags as potentially weaker due to unbalanced ionization-mode sampling

## Limitations

- The pair sampling balanced Tanimoto bins but did not enforce balanced sampling of ionization-mode pairs, which may limit negative vs. negative mode prediction performance
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to the training data
- Fragment peak binning may lose useful information, and unequal pair counts per Tanimoto bin per ionization mode can skew bin-level statistics
- Risk that the model is not well-suited for very specific tasks or chemical classes

## Evidence

- [abstract] cross-mode correlation claim: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [intro] metadata input improves prediction: "using ionization mode and precursor m/z as input improves model performance"
- [discussion] uncertainty filtering raises accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [readme] model supports cross-mode predictions: [evidence span withheld — anchor status: external_doc]
- [readme] similarity matrix output: [evidence span withheld — anchor status: external_doc]
