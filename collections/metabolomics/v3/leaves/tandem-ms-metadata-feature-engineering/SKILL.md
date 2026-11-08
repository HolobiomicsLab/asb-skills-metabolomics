---
name: tandem-ms-metadata-feature-engineering
description: 'Decision support for adding spectral metadata as model input in MS2DeepScore
  2.0 training: build CategoricalToBinary/StandardScaler features from precursor m/z,
  ionmode, adduct, or instrument type and test whether each improves RMSE.'
when_to_use_negative:
- Input spectra lack reliable adduct annotations — the article excludes adduct from
  the default model precisely because 'not all preprocessing tools generate reliable
  adduct information', and incorrect adduct data can reduce performance
- You only have one ionization mode; the cross-mode benefit hinges on ionisation_mode='both'
  training data and ionmode encoding
- Working with MS1 spectra — public spectral libraries generally lack annotated raw
  MS1 spectra, so this feature engineering is not applicable there
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Siamese network trained on paired MS/MS spectra with metadata-augmented input
    features to predict Tanimoto chemical similarities
  repo: https://github.com/matchms/MS2DeepScore
- name: PyTorch
  role: Framework implementing the Siamese model architecture and training loop
- name: ms2deepscore
  role: Python package providing SettingsMS2Deepscore, pair sampling, and train_ms2deepscore_wrapper
    used to configure and run the metadata-feature training
  repo: https://pypi.org/project/ms2deepscore
- name: mzmine
  role: Local deployment path that runs MS2DeepScore molecular networking so scientists
    without programming expertise can consume the similarity scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:f3ab2e5c024c16939a547ce7f3d2aa2c734e28d059af9744923197b48ec0393b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - SettingsMS2Deepscore
  - ms2deepscore
  - mzmine
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Tandem-MS metadata feature engineering

## Summary

Encodes tandem-MS metadata (precursor m/z, ionization mode, adduct, instrument type) as numeric input features for a Siamese spectral similarity network, then ablates which encodings actually improve Tanimoto-similarity prediction accuracy.

## When to use

Decision support for adding spectral metadata as model input in MS2DeepScore 2.0 training: build CategoricalToBinary/StandardScaler features from precursor m/z, ionmode, adduct, or instrument type and test whether each improves RMSE.

## When NOT to use

- Input spectra lack reliable adduct annotations — the article excludes adduct from the default model precisely because 'not all preprocessing tools generate reliable adduct information', and incorrect adduct data can reduce performance
- You only have one ionization mode; the cross-mode benefit hinges on ionisation_mode='both' training data and ionmode encoding
- Working with MS1 spectra — public spectral libraries generally lack annotated raw MS1 spectra, so this feature engineering is not applicable there

## Inputs

- Annotated tandem mass spectra in mgf/msp/mzML formats with metadata fields precursor_mz, ionmode, adduct, instrument type
- Reference chemical structures (InChI/SMILES) for computing Tanimoto similarity training labels
- SettingsMS2Deepscore configuration specifying spectrum_file_path, additional_metadata, and ionisation_mode='both'

## Outputs

- Trained MS2DeepScore 2.0 Siamese network variants per metadata configuration
- RMSE between predicted and reference Tanimoto similarities per model variant
- ms2deepscore_model.pt checkpoint plus embedding evaluator if SettingsEmbeddingEvaluator is passed

## How to apply

Load annotated MS/MS spectra covering both ionization modes with per-spectrum metadata (precursor m/z, ionmode, adduct, instrument type) plus reference structures whose Tanimoto scores serve as training labels. Define encodings in SettingsMS2Deepscore's additional_metadata list: ionmode via CategoricalToBinary (positive→1, negative→0), precursor_mz via StandardScaler (mean 0, standard deviation 1000). Train the Siamese PyTorch network with the new pair sampling algorithm that balances spectrum pairs over Tanimoto similarity bins, comparing a baseline model without metadata against variants adding precursor m/z + ionmode, +adduct, and one-hot instrument type. Evaluate each variant on held-out test pairs by RMSE between predicted and reference Tanimoto scores; the ablation tells you which metadata genuinely carries signal. Instrument-type one-hot encoding gave no improvement, so keep it out; adduct was excluded from the default model because preprocessing tools do not always produce reliable adduct annotations.

## Related tools

- **MS2DeepScore 2.0** (Siamese network trained on paired MS/MS spectra with metadata-augmented input features to predict Tanimoto chemical similarities) — https://github.com/matchms/MS2DeepScore
- **PyTorch** (Framework implementing the Siamese model architecture and training loop)
- **ms2deepscore** (Python package providing SettingsMS2Deepscore, pair sampling, and train_ms2deepscore_wrapper used to configure and run the metadata-feature training) — https://pypi.org/project/ms2deepscore
- **mzmine** (Local deployment path that runs MS2DeepScore molecular networking so scientists without programming expertise can consume the similarity scores)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; settings = SettingsMS2Deepscore(spectrum_file_path="./combined_libraries.mgf", additional_metadata=[("CategoricalToBinary", {"metadata_field": "ionmode", "entries_becoming_one": "positive", "entries_becoming_zero": "negative"}), ("StandardScaler", {"metadata_field": "precursor_mz", "mean": 0, "standard_deviation": 1000})], ionisation_mode="both"); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- RMSE on held-out test pairs must decrease (or stay equal) when precursor m/z + ionmode features are added relative to the no-metadata baseline
- One-hot instrument-type variant should show no improvement over baseline — if it does, the ablation was configured wrongly
- Pair sampling must balance spectrum pairs over Tanimoto bins and molecule sampling frequency; inspect the pair-sampling distribution before training
- Embedding evaluator should flag spectra with low information content or dissimilarity to training data; high predicted MSE spectra should be excluded before trusting similarity scores

## Limitations

- One-hot encoding of instrument type does not improve model performance
- Adduct type excluded from default model because preprocessing tools do not reliably generate adduct information
- Public mass spectral libraries generally lack annotated raw MS1 spectra, so MS1 features cannot be included in training
- Fragment signals are initially binned, which might result in loss of useful information
- Poor predictions expected for low-quality spectra with limited fragmentation or chimeric spectra from multiple precursor ions
- The new sampling algorithm did not enforce balanced sampling of different ionization mode pairs, possibly limiting negative-vs-negative mode predictions

## Evidence

- [intro] Metadata encoding drives the ablation design: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [readme] README encodes the concrete feature engineering: [evidence span withheld — anchor status: external_doc]
- [discussion] Instrument-type one-hot is a no-op: [evidence span withheld — anchor status: not_found]
- [discussion] Adduct reliability caveat: "not all preprocessing tools generate reliable adduct information. Therefore, we decided to not include this in the default model"
- [intro] Pair sampling algorithm reduces training bias: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [abstract] Quality flagging protects downstream users: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
