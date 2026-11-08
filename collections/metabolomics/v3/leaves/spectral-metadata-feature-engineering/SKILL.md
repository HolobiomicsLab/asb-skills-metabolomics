---
name: spectral-metadata-feature-engineering
description: Use when training or applying MS2DeepScore 2.0 on tandem mass spectra
  in both positive and negative ionization modes and the model must distinguish ionization
  mode and precursor m/z context from peak data alone.
when_to_use_negative:
- Do not add adduct information to the model input when preprocessing tools cannot
  guarantee reliable adduct annotations, as the article explicitly excluded adduct
  from the default model for this reason
- Skip this skill when only MS1-level data are available; the article notes public
  mass spectral libraries generally lack annotated raw MS1 spectra, so this metadata
  engineering targets MS/MS spectra only
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore 2.0
  role: Consumes the engineered metadata features (ionization mode one-hot and scaled
    precursor m/z) as additional inputs to its Siamese network alongside fragment
    intensities
  repo: https://github.com/matchms/MS2DeepScore
- name: PyTorch
  role: Framework in which the Siamese model that ingests the metadata-augmented features
    is implemented and trained
- name: matchms
  role: Recommended cleaning pipeline that prepares spectra and their metadata fields
    before they are fed into MS2DeepScore training
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:e249af349629a0c3fd7ad21291e7cf68e755520d6e2ed463d6336233faf59c7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-metadata-feature-engineering

## Summary

Augment tandem MS peak intensities with spectral metadata (precursor m/z and ionization mode) as additional inputs to the MS2DeepScore 2.0 Siamese network, which the article shows improves chemical-similarity prediction quality, especially for cross-ionization-mode pairs.

## When to use

Use when training or applying MS2DeepScore 2.0 on tandem mass spectra in both positive and negative ionization modes and the model must distinguish ionization mode and precursor m/z context from peak data alone.

## When NOT to use

- Do not add adduct information to the model input when preprocessing tools cannot guarantee reliable adduct annotations, as the article explicitly excluded adduct from the default model for this reason
- Skip this skill when only MS1-level data are available; the article notes public mass spectral libraries generally lack annotated raw MS1 spectra, so this metadata engineering targets MS/MS spectra only

## Inputs

- Binned tandem MS (MS/MS) spectra with precursor m/z and ionization mode metadata (mgf, msp, mzml, mzxml, json, usi formats supported)
- Spectrum libraries such as GNPS, MassBank EU, MoNA, and MSnLib with known chemical structures for Tanimoto labels
- SettingsMS2Deepscore configuration specifying additional_metadata transformations and ionisation_mode

## Outputs

- Metadata-augmented training features (binary ionization-mode flag and scaled precursor m/z) paired with binned fragment-intensity vectors
- Trained MS2DeepScore 2.0 Siamese model that predicts Tanimoto similarity across same-mode and cross-mode spectrum pairs

## How to apply

Attach precursor m/z and ionization mode metadata to each binned tandem MS spectrum from GNPS, MassBank EU, MoNA, and MSnLib, then pass these metadata as additional model inputs alongside the fragment-intensity vectors when training the Siamese network in PyTorch. In the ms2deepscore code base this is done by supplying additional_metadata entries to SettingsMS2Deepscore: a CategoricalToBinary encoder maps the 'ionmode' metadata field to a binary feature (positive entries become one, negative become zero), and a StandardScaler transforms 'precursor_mz' with a normalization mean of 0 and standard deviation of 1000. Set ionisation_mode='both' so the model trains on spectra from both ionization modes rather than a single mode. The article reports that using ionization mode and precursor m/z as model input improves prediction quality; do not include adduct information by default, since not all preprocessing tools generate reliable adduct annotations and incorrect adduct data can reduce performance.

## Related tools

- **MS2DeepScore 2.0** (Consumes the engineered metadata features (ionization mode one-hot and scaled precursor m/z) as additional inputs to its Siamese network alongside fragment intensities) — https://github.com/matchms/MS2DeepScore
- **PyTorch** (Framework in which the Siamese model that ingests the metadata-augmented features is implemented and trained)
- **matchms** (Recommended cleaning pipeline that prepares spectra and their metadata fields before they are fed into MS2DeepScore training) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', additional_metadata=[('CategoricalToBinary', {'metadata_field':'ionmode','entries_becoming_one':'positive','entries_becoming_zero':'negative'}), ('StandardScaler', {'metadata_field':'precursor_mz','mean':0,'standard_deviation':1000})], ionisation_mode='both')
```

## Evaluation signals

- Confirm that SettingsMS2Deepscore additional_metadata contains a CategoricalToBinary entry for 'ionmode' (positive→1, negative→0) and a StandardScaler entry for 'precursor_mz' with mean=0 and standard_deviation=1000
- Verify the training settings set ionisation_mode='both' so the model receives spectra from positive and negative ionization modes
- Check that predicted vs. true Tanimoto similarity for cross-ionization-mode pairs on the held-out test set shows a strong correlation, consistent with Figure 2b
- Ensure no adduct metadata is silently added to the model input in the default configuration

## Limitations

- Not all preprocessing tools generate reliable adduct information, so adduct is excluded from the default model; incorrect adduct info could reduce performance
- Fragment binning may lose useful information that metadata features cannot recover
- The new pair-sampling algorithm did not enforce balanced sampling of different ionization-mode pairs, which might limit performance for negative vs. negative predictions
- Poor predictions are expected for low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to the training data

## Evidence

- [readme] Metadata transformations in ms2deepscore settings: [evidence span withheld — anchor status: external_doc]
- [intro] Article claim that metadata input improves performance: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [discussion] Exclusion of adduct metadata from the default model: "not all preprocessing tools generate reliable adduct information. Therefore, we decided to not include this in the default model"
- [intro] Model trained on both ionization modes: "MS2DeepScore 2.0 models are trained on mass fragmentation spectra in both ionization modes"
- [readme] Training wrapper with ionisation_mode setting: [evidence span withheld — anchor status: external_doc]
