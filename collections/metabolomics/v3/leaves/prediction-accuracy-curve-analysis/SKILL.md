---
name: prediction-accuracy-curve-analysis
description: Use when you have per-spectrum embedding-quality estimates (predicted
  MSE from an embedding evaluator) and a similarity prediction error metric, and want
  to quantify how much accuracy improves as the highest-uncertainty spectra are removed.
when_to_use_negative:
- Spectra have no per-spectrum uncertainty estimates (no embedding evaluator trained
  alongside the model), so there is no predicted MSE to rank by
- Reference Tanimoto similarities are unavailable for the test spectra, so accuracy
  cannot be recomputed after each removal step
- The goal is a single similarity matrix for molecular networking rather than diagnosing
  how accuracy depends on spectrum quality
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore 2.0
  role: computes embeddings and predicted Tanimoto similarities for all pairs of test
    spectra, whose accuracy is tracked along the filtering curve
  repo: github.com/matchms/MS2DeepScore
- name: PyTorch
  role: framework in which the MS2DeepScore 2.0 model and InceptionTime embedding
    evaluator are implemented and run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:70fa9b8bd44728f79a9370a4c139462919a7f9938efb9e3558e994be5aeb7626
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# prediction-accuracy-curve-analysis

## Summary

Builds a prediction-accuracy curve for MS2DeepScore 2.0 by iteratively removing test spectra with the highest predicted MSE from the InceptionTime embedding evaluator and recomputing prediction accuracy (e.g., RMSE between predicted and true Tanimoto similarity) on the remaining spectra, demonstrating that overall accuracy rises as unreliable spectra are filtered out (Figure 4c).

## When to use

Use when you have per-spectrum embedding-quality estimates (predicted MSE from an embedding evaluator) and a similarity prediction error metric, and want to quantify how much accuracy improves as the highest-uncertainty spectra are removed.

## When NOT to use

- Spectra have no per-spectrum uncertainty estimates (no embedding evaluator trained alongside the model), so there is no predicted MSE to rank by
- Reference Tanimoto similarities are unavailable for the test spectra, so accuracy cannot be recomputed after each removal step
- The goal is a single similarity matrix for molecular networking rather than diagnosing how accuracy depends on spectrum quality

## Inputs

- Pre-trained MS2DeepScore 2.0 model file (e.g., ms2deepscore_model.pt from Zenodo record 17826815)
- Test set of tandem mass spectra with reference Tanimoto similarities (positive and negative ionization modes; formats such as mgf, msp, mzml supported)
- Per-spectrum predicted MSE values from the InceptionTime embedding evaluator

## Outputs

- Removal-fraction vs prediction-accuracy table (figure4c_mse_filtering.csv)
- Prediction-accuracy curve showing accuracy improving as highest-predicted-MSE spectra are removed (Figure 4c analogue)

## How to apply

Load the pre-trained MS2DeepScore 2.0 model with its MS2DeepScoreEvaluated embedding-evaluator wrapper and a test set of tandem mass spectra with reference Tanimoto similarities, spanning both positive and negative ionization modes. Compute MS2DeepScore embeddings and predicted Tanimoto similarities for all pairs of test spectra, then run the InceptionTime embedding evaluator on each embedding to obtain a per-spectrum predicted MSE (an embedding-quality estimate). Rank spectra by predicted MSE in descending order and iteratively remove the highest-MSE spectra in fixed fractions of the test set; after each removal step recompute prediction accuracy (e.g., RMSE between predicted and true Tanimoto similarity) on the remaining spectra. The rationale is that high predicted MSE flags unreliable embeddings caused by low spectral quality, limited fragmentation, chimeric spectra, or spectra dissimilar to the training data, so removing them should raise overall accuracy. Write the removal-fraction vs accuracy table to figure4c_mse_filtering.csv and verify the improving-accuracy trend reproduces Figure 4c.

## Related tools

- **MS2DeepScore 2.0** (computes embeddings and predicted Tanimoto similarities for all pairs of test spectra, whose accuracy is tracked along the filtering curve) — github.com/matchms/MS2DeepScore
- **PyTorch** (framework in which the MS2DeepScore 2.0 model and InceptionTime embedding evaluator are implemented and run)

## Examples

```
from ms2deepscore import MS2DeepScore; ms2ds_model = MS2DeepScore(model); embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- Prediction accuracy (RMSE between predicted and true Tanimoto similarity) on the remaining spectra must increase monotonically (or near-monotonically) as the highest-predicted-MSE spectra are removed, reproducing the Figure 4c trend
- The removal fractions must partition the test set in fixed fractions ranked by descending predicted MSE
- Per-spectrum predicted MSE must be defined for every spectrum in both positive and negative ionization modes before ranking
- The removal-fraction vs accuracy table should be written to figure4c_mse_filtering.csv with one accuracy value per removal step

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions, or spectra dissimilar to training data, which drives the high-MSE tail
- Filtering out spectra discards data points rather than improving the model itself; accuracy gains apply only to the remaining spectra
- The pair sampling algorithm did not enforce balanced sampling of the different ionization mode pairs, which may limit negative vs. negative mode prediction performance

## Evidence

- [discussion] Removing test spectra with high predicted uncertainty to raise overall accuracy: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [intro] Filtering unreliable spectra: "This allows users to filter out spectra for which the MS2DeepScore predictions are unreliable, e.g., due to low spectral quality or when spectra differ substantially from the training data."
- [abstract] Embedding evaluator flags low-quality spectra: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [readme] README embedding evaluator availability: [evidence span withheld — anchor status: external_doc]
- [discussion] Poor prediction cases: "Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric mass spectra from multiple precursor ions"
