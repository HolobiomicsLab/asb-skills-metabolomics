---
name: deep-learning-input-ablation-analysis
description: Use when a trained deep learning model has uncertain which inputs contribute
  to performance, and you can define parallel model variants differing only in the
  input features to compare held-out metrics such as RMSE.
when_to_use_negative:
- Do not include adduct type as an ablated input feature when preprocessing tools
  cannot supply reliable adduct annotations — incorrect adduct information could reduce
  performance, which is why the default MS2DeepScore model excludes it
- Do not infer that a metadata field will help just because it is conceptually related
  — one-hot encoded instrument type did not improve performance, so the ablation,
  not intuition, must decide
- Do not compare variants trained with different pair-sampling schemes; biased training-pair
  distributions confound the ablation
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network model being ablated over metadata inputs (precursor
    m/z, ionization mode, adduct type, instrument type)
  repo: https://github.com/matchms/MS2DeepScore
- name: ms2deepscore
  role: Python library providing SettingsMS2Deepscore for configuring additional_metadata
    per variant and the training pipeline
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the Siamese network variants are implemented and trained
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:78dbbfa3ca0c1791da9f7e1acdd5ef7cdc87ea8d050c306ef1ddedb4752ed4da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - SettingsMS2Deepscore
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Deep learning input ablation analysis

## Summary

Systematically add or remove input feature groups (e.g. spectral metadata such as precursor m/z, ionization mode, adduct type, instrument type) from a neural network and compare each variant's held-out evaluation metric to isolate which inputs actually drive predictive performance.

## When to use

Use when a trained deep learning model has uncertain which inputs contribute to performance, and you can define parallel model variants differing only in the input features to compare held-out metrics such as RMSE.

## When NOT to use

- Do not include adduct type as an ablated input feature when preprocessing tools cannot supply reliable adduct annotations — incorrect adduct information could reduce performance, which is why the default MS2DeepScore model excludes it
- Do not infer that a metadata field will help just because it is conceptually related — one-hot encoded instrument type did not improve performance, so the ablation, not intuition, must decide
- Do not compare variants trained with different pair-sampling schemes; biased training-pair distributions confound the ablation

## Inputs

- Annotated tandem mass spectra training dataset (both positive and negative ionization modes) with per-spectrum metadata: precursor m/z, ionization mode, adduct type, instrument type
- Reference chemical structures (SMILES/InChI) for computing Tanimoto similarity labels
- Model configuration defining variant variants, e.g. ms2deepscore SettingsMS2Deepscore with additional_metadata list

## Outputs

- One trained Siamese neural network (e.g. .pt checkpoint) per input-ablation variant
- Per-variant RMSE between predicted and reference Tanimoto similarities on held-out test pairs
- Comparison table, e.g. metadata_input_ablation.csv, with one row per model variant and columns for the metadata configuration and evaluation score

## How to apply

Define model variants that differ only in input features: a baseline using only the spectra (e.g. binned MS/MS peaks), variants adding scaled precursor m/z and binary-encoded ionization mode via ms2deepscore's SettingsMS2Deepscore additional_metadata (CategoricalToBinary for ionmode, StandardScaler for precursor_mz), and variants adding one-hot encoded adduct type or instrument type. Train each Siamese network variant in PyTorch with identical settings, using the pair sampling algorithm that balances spectrum pairs over Tanimoto similarity bins so comparisons are fair. Evaluate each variant on held-out test spectrum pairs by computing RMSE between predicted and reference Tanimoto chemical similarities, then compile per-variant scores into a comparison table (e.g. metadata_input_ablation.csv with one row per variant). Interpretation is ablation-by-addition: only feature groups whose variant beats the baseline RMSE are judged useful; in the source study, precursor m/z + ionization mode (and adduct type) improved MS2DeepScore performance while one-hot encoded instrument type did not.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network model being ablated over metadata inputs (precursor m/z, ionization mode, adduct type, instrument type)) — https://github.com/matchms/MS2DeepScore
- **ms2deepscore** (Python library providing SettingsMS2Deepscore for configuring additional_metadata per variant and the training pipeline) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the Siamese network variants are implemented and trained)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field': 'ionmode', 'entries_becoming_one': 'positive', 'entries_becoming_zero': 'negative'}), ('StandardScaler', {'metadata_field': 'precursor_mz', 'mean': 0, 'standard_deviation': 1000})], ionisation_mode='both')
```

## Evaluation signals

- Variant using precursor m/z + ionization mode achieves lower held-out RMSE than the metadata-free baseline
- One-hot encoded instrument-type variant does NOT improve RMSE over baseline (negative control matching the reported result)
- All variants are trained with the pair sampling algorithm that balances spectrum pairs over Tanimoto similarity bins, so differences reflect inputs, not sampling
- Output table contains exactly one row per defined variant with a metadata-configuration column and an evaluation-score column

## Limitations

- Adduct-type input was excluded from the default model because not all preprocessing tools generate reliable adduct information; incorrect adduct info could reduce performance
- The new pair sampling algorithm balances pairs over Tanimoto bins and molecule sampling frequency but did not enforce balanced sampling of the different ionization mode pairs, which may limit negative-vs-negative mode comparisons
- Results are specific to the MS2DeepScore architecture and training data (>500,000 MS/MS spectra from GNPS, Mona, MassBank, MSnLib); ablation conclusions may not transfer to smaller or differently distributed datasets

## Evidence

- [intro] intro: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [discussion] discussion: "not all preprocessing tools generate reliable adduct information. Therefore, we decided to not include this in the default model"
- [readme] readme: [evidence span withheld — anchor status: not_found]
