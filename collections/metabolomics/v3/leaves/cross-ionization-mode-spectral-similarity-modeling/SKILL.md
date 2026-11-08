---
name: cross-ionization-mode-spectral-similarity-modeling
description: Use when positive- and negative-ionization-mode MS/MS spectra must be
  compared in one unified analysis, or when evaluating whether spectral metadata (precursor
  m/z, ionmode, adduct, instrument type) improves similarity prediction.
when_to_use_negative:
- Your preprocessing tools do not produce reliable adduct annotations and you intend
  to feed adduct type as model input — incorrect adduct information can reduce performance
  and it is excluded from the default model
- Spectra are low-quality with limited fragmentation, chimeric (multiple precursor
  ions), or dissimilar to the training data — predictions are unreliable; filter them
  using the embedding evaluator
- You only have a small custom dataset and plan to train from scratch — instead merge
  it with larger public libraries or fine-tune the pre-trained model
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore 2.0
  role: Siamese neural network that predicts Tanimoto chemical similarity between
    spectra, including across ionization modes
  repo: github.com/matchms/MS2DeepScore
- name: ms2deepscore
  role: Python library providing SettingsMS2Deepscore, pair sampling, training wrapper,
    and the MS2DeepScore scoring wrapper
  repo: github.com/matchms/ms2deepscore
- name: PyTorch
  role: Backend framework in which the entire Siamese model is implemented
- name: matchms
  role: Provides the spectrum cleaning/filtering Pipeline (DEFAULT_FILTERS) and Pipeline-based
    score computation before and during similarity prediction
  repo: https://github.com/matchms/matchms
- name: UMAP
  role: Dimensionality reduction for visualizing MS2DeepScore embeddings in chemical
    space
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:3494c6da29936f02686e574ce36a20746ee38e36936e8e8501f64e1e321c116a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - SettingsMS2Deepscore
  - UMAP
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-ionization-mode-spectral-similarity-modeling

## Summary

Train or apply MS2DeepScore 2.0, a PyTorch Siamese neural network that predicts Tanimoto chemical similarity between tandem MS/MS spectra from both the same and different ionization modes, optionally augmented with metadata inputs (precursor m/z, ionization mode).

## When to use

Use when positive- and negative-ionization-mode MS/MS spectra must be compared in one unified analysis, or when evaluating whether spectral metadata (precursor m/z, ionmode, adduct, instrument type) improves similarity prediction.

## When NOT to use

- Your preprocessing tools do not produce reliable adduct annotations and you intend to feed adduct type as model input — incorrect adduct information can reduce performance and it is excluded from the default model
- Spectra are low-quality with limited fragmentation, chimeric (multiple precursor ions), or dissimilar to the training data — predictions are unreliable; filter them using the embedding evaluator
- You only have a small custom dataset and plan to train from scratch — instead merge it with larger public libraries or fine-tune the pre-trained model

## Inputs

- Annotated tandem MS/MS spectra (mgf, msp, mzML, mzXML, JSON, USI) from positive and negative ionization modes with per-spectrum metadata (precursor m/z, ionmode, adduct type, instrument type)
- Reference molecular structures for computing Tanimoto similarity labels
- Pre-trained ms2deepscore_model.pt checkpoint from Zenodo (Zenodo record 17826815)
- Cleaned spectra produced by the matchms DEFAULT_FILTERS pipeline

## Outputs

- Pairwise chemical similarity matrix (numpy array of MS2DeepScore scores between all spectra)
- Spectrum embedding vectors (embedding array for UMAP visualization)
- Trained Siamese model file (ms2deepscore_model.pt) plus optional embedding evaluator
- Per-variant RMSE comparison of metadata-input ablation (e.g. metadata_input_ablation.csv)

## How to apply

Clean spectra with the matchms filtering pipeline, then train the Siamese network via SettingsMS2Deepscore with ionisation_mode='both' and additional_metadata transformers such as CategoricalToBinary for ionmode and StandardScaler for precursor_mz (mean 0, standard deviation 1000). Use the pair sampling algorithm that balances spectrum pairs over Tanimoto similarity bins, and check pair sampling quality for new datasets since it can be suboptimal if not re-optimized. To run the metadata ablation, compare model variants (baseline without metadata; precursor m/z + ionization mode; plus adduct type; plus one-hot instrument type) by RMSE between predicted and reference Tanimoto similarities on held-out pairs. For prediction, load the pre-trained ms2deepscore_model.pt (>500,000 MS/MS spectra from GNPS, MoNA, MassBank, MSnLib), run MS2DeepScore through a matchms Pipeline over mgf/msp/mzml inputs, and extract the similarity matrix or embedding vectors for UMAP visualization.

## Related tools

- **MS2DeepScore 2.0** (Siamese neural network that predicts Tanimoto chemical similarity between spectra, including across ionization modes) — github.com/matchms/MS2DeepScore
- **ms2deepscore** (Python library providing SettingsMS2Deepscore, pair sampling, training wrapper, and the MS2DeepScore scoring wrapper) — github.com/matchms/ms2deepscore
- **PyTorch** (Backend framework in which the entire Siamese model is implemented)
- **matchms** (Provides the spectrum cleaning/filtering Pipeline (DEFAULT_FILTERS) and Pipeline-based score computation before and during similarity prediction) — https://github.com/matchms/matchms
- **UMAP** (Dimensionality reduction for visualizing MS2DeepScore embeddings in chemical space)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path="combined_libraries.mgf", ionisation_mode="both")
```

## Evaluation signals

- Metadata-ablation invariant: variants using precursor m/z + ionization mode (and adduct type) achieve lower RMSE than the no-metadata baseline, while the one-hot instrument-type variant does not improve over baseline
- Cross-mode invariant: the trained or loaded model produces valid similarity predictions between spectra of the same AND different ionization modes (both positive-positive, negative-negative, and positive-negative pairs)
- Similarity matrix shape check: pipeline.scores.to_array() yields an N x N numpy array for N query spectra
- Pair sampling check: spectrum pairs are balanced over Tanimoto similarity bins before training; re-optimize sampling if the distribution is skewed
- Embedding evaluator flags spectra with high predicted uncertainty/low information content; removing high-uncertainty test spectra should raise overall accuracy

## Limitations

- The new pair sampling algorithm did not enforce balanced sampling of the different ionization mode pairs, which may have limited negative-vs-negative mode prediction performance
- Adduct input is excluded from the default model because not all preprocessing tools generate reliable adduct information, and incorrect adduct info could reduce performance
- MS1 raw spectra are not included in training because public mass spectral libraries generally lack annotated raw MS1 spectra; fragment binning may also discard useful information
- Poor predictions are expected for low-quality spectra with limited fragmentation or chimeric mass spectra from multiple precursor ions, and the model risks being unsuited to very specific tasks or chemical classes

## Evidence

- [intro] metadata improves performance: "we explore and evaluate the addition of metadata to the model input and show that using ionization mode and precursor m/z as input improves model performance"
- [abstract] model definition: "we present MS2DeepScore 2.0, a machine learning-based model to predict chemical similarity between mass fragmentation spectra"
- [intro] training scope: "MS2DeepScore 2.0 models are trained on mass fragmentation spectra in both ionization modes"
- [readme] README model description: "provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) "
- [readme] README pre-trained model scope: [evidence span withheld — anchor status: external_doc]
- [discussion] pair sampling caveat: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] adduct exclusion rationale: "not all preprocessing tools generate reliable adduct information. Therefore, we decided to not include this in the default model"
