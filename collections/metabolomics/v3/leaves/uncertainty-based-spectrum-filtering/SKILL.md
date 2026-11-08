---
name: uncertainty-based-spectrum-filtering
description: Use when MS2DeepScore-predicted Tanimoto similarities contain unreliable
  spectra (low information content, chimeric spectra, or spectra dissimilar to training
  data) and you need to filter them using per-spectrum embedding-quality estimates.
when_to_use_negative:
- The MS2DeepScore model you use was trained without an embedding evaluator (SettingsEmbeddingEvaluator
  left as None), so no per-spectrum predicted MSE is available.
- All input spectra are known to closely match the training data with high spectral
  quality, so uncertainty filtering would remove informative spectra without benefit.
- You need similarities for a model trained on only one ionization mode (e.g., DreaMS,
  trained only on positive-mode spectra), where the cross-mode quality-estimation
  setup does not apply.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore 2.0
  role: Provides the Siamese-network embeddings and the MS2DeepScoreEvaluated embedding-evaluator
    whose predicted MSE drives the uncertainty ranking and filtering.
  repo: github.com/matchms/MS2DeepScore
- name: PyTorch
  role: Implementation framework in which the model and embedding evaluator are run.
- name: matchms
  role: Cleaning/filtering pipeline (DEFAULT_FILTERS) applied to spectra before embedding
    and uncertainty estimation.
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:27954613f9c3489414a3dd9b025043e39508d771c290909029167ea6b4d507df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# uncertainty-based-spectrum-filtering

## Summary

Ranks tandem mass spectra by a per-spectrum predicted MSE from the InceptionTime embedding evaluator wrapped around MS2DeepScore 2.0, then iteratively removes the highest-uncertainty spectra to raise the accuracy of predicted Tanimoto similarities on the retained spectra.

## When to use

Use when MS2DeepScore-predicted Tanimoto similarities contain unreliable spectra (low information content, chimeric spectra, or spectra dissimilar to training data) and you need to filter them using per-spectrum embedding-quality estimates.

## When NOT to use

- The MS2DeepScore model you use was trained without an embedding evaluator (SettingsEmbeddingEvaluator left as None), so no per-spectrum predicted MSE is available.
- All input spectra are known to closely match the training data with high spectral quality, so uncertainty filtering would remove informative spectra without benefit.
- You need similarities for a model trained on only one ionization mode (e.g., DreaMS, trained only on positive-mode spectra), where the cross-mode quality-estimation setup does not apply.

## Inputs

- Test set of tandem mass spectra (MGF/MSP/mzML/mzXML/JSON/USI) spanning positive and negative ionization modes
- Reference (true) Tanimoto similarities between test spectra pairs
- Pre-trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt, e.g., from Zenodo record 17826815) with embedding-evaluator wrapper

## Outputs

- Per-spectrum predicted MSE values (embedding-quality estimates)
- Removal-fraction vs accuracy (RMSE) table, e.g., figure4c_mse_filtering.csv
- Filtered set of spectra with improved MS2DeepScore prediction accuracy

## How to apply

Load the pre-trained MS2DeepScore 2.0 model with its MS2DeepScoreEvaluated embedding-evaluator wrapper and a test set of tandem mass spectra (positive and negative ionization modes) with reference Tanimoto similarities. Compute MS2DeepScore embeddings and all pairwise predicted Tanimoto similarities, then run the InceptionTime embedding evaluator on each embedding to obtain a per-spectrum predicted MSE as an embedding-quality estimate. Rank spectra by predicted MSE in descending order and iteratively remove the highest-MSE spectra in fixed fractions of the test set; after each removal step, recompute prediction accuracy as the RMSE between predicted and true Tanimoto similarities on the remaining spectra. The rationale is that spectra with high predicted MSE (low spectral quality, limited fragmentation, or out-of-distribution spectra) drive false predictions, so dropping them raises overall accuracy, as shown in Figure 4c. Write the removal-fraction vs accuracy table to figure4c_mse_filtering.csv.

## Related tools

- **MS2DeepScore 2.0** (Provides the Siamese-network embeddings and the MS2DeepScoreEvaluated embedding-evaluator whose predicted MSE drives the uncertainty ranking and filtering.) — github.com/matchms/MS2DeepScore
- **PyTorch** (Implementation framework in which the model and embedding evaluator are run.)
- **matchms** (Cleaning/filtering pipeline (DEFAULT_FILTERS) applied to spectra before embedding and uncertainty estimation.) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore.models import load_model; from ms2deepscore import MS2DeepScore; ms2ds = MS2DeepScore(load_model("ms2deepscore_model.pt")); embeddings = ms2ds.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- RMSE between predicted and true Tanimoto similarities on the remaining spectra should increase (accuracy improves) monotonically as the highest-predicted-MSE spectra are removed, reproducing the Figure 4c trend.
- Per-spectrum predicted MSE values must be produced for every input spectrum in both positive and negative ionization modes.
- The removal fractions must span fixed fractions of the test set, and each accuracy value must be computed only on the spectra remaining after removal.
- The output CSV (figure4c_mse_filtering.csv) must contain a removal-fraction column paired with an accuracy/RMSE column.
- Spectra flagged with highest MSE should correspond to low-information-content or training-distant spectra rather than random spectra.

## Limitations

- Poor predictions are expected for low-quality spectra with limited fragmentation and chimeric mass spectra from multiple precursor ions, so filtering thresholds may need adjustment per dataset.
- Spectra differing substantially from the training data are flagged, but the model may still be poorly suited for very specific tasks or chemical classes even after filtering.
- The embedding evaluator must have been trained together with the model (SettingsEmbeddingEvaluator); models trained without it cannot support this skill.
- Pair sampling was not balanced over ionization modes, which may affect negative-vs-negative mode prediction performance for the pairs being filtered.

## Evidence

- [intro] filter-out-unreliable: "This allows users to filter out spectra for which the MS2DeepScore predictions are unreliable, e.g., due to low spectral quality or when spectra differ substantially from the training data."
- [discussion] accuracy-raise: "the overall accuracy can be raised by removing the test spectra with a high predicted uncertainty, see Figure 4"
- [abstract] flags-low-information: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [readme] readme-embedding-evaluator: [evidence span withheld — anchor status: external_doc]
- [readme] readme-evaluator-training: [evidence span withheld — anchor status: external_doc]
