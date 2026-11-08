---
name: spectrum-embedding-extraction-from-transformer
description: Use when you need internal model states — embeddings, attention weights,
  predictions — from a trained LipiDetective transformer over preprocessed tandem
  MS2 lipid spectra, to feed interpretability or validation analyses.
when_to_use_negative:
- You want to identify lipids directly from raw mzML files without first running LipiDetective's
  preprocessing (smoothing, peak picking, normalization) — the transformer expects
  preprocessed HDF5 spectra
- You only need surface-level lipid identifications and have no interest in model
  internals or interpretability — running the full inference pipeline without embedding
  extraction is simpler
- Your spectra are outside the trained domain (e.g. not lipid tandem MS2 spectra)
  — the model learned characteristic lipid fragmentation patterns and does not generalize
  to arbitrary MS2 spectra
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model whose spectrum embeddings and attention weights are extracted
    during inference over HDF5 MS2 lipid spectra
  repo: github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Framework in which LipiDetective's transformer module was implemented and
    from which inference is run
- name: h5py
  role: Library used to load preprocessed MS2 spectra from HDF5 files for model input
- name: Captum
  role: Library through which integrated gradients gradients over the extracted embeddings
    are computed for downstream interpretability
- name: pyOpenMS
  role: Library used for preprocessing raw spectra (smoothing, baseline correction,
    peak picking) into model-ready form
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:9cecdb1cb71c7f133e990def6d82c923b6ee59e9e2f4ebb8d47ab7e5f9d4e1a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Captum
  - LipiDetective
  - PyTorch
  - h5py
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectrum-embedding-extraction-from-transformer

## Summary

Extracts per-spectrum embeddings and attention weights from a trained LipiDetective transformer model so that downstream interpretability analyses (e.g. integrated gradients peak attribution) and prediction introspection can be performed on MS2 lipid spectra stored as HDF5.

## When to use

Use when you need internal model states — embeddings, attention weights, predictions — from a trained LipiDetective transformer over preprocessed tandem MS2 lipid spectra, to feed interpretability or validation analyses.

## When NOT to use

- You want to identify lipids directly from raw mzML files without first running LipiDetective's preprocessing (smoothing, peak picking, normalization) — the transformer expects preprocessed HDF5 spectra
- You only need surface-level lipid identifications and have no interest in model internals or interpretability — running the full inference pipeline without embedding extraction is simpler
- Your spectra are outside the trained domain (e.g. not lipid tandem MS2 spectra) — the model learned characteristic lipid fragmentation patterns and does not generalize to arbitrary MS2 spectra

## Inputs

- Preprocessed MS2 lipid spectra stored in HDF5 format (m/z range 50–1600, base peak normalized)
- Trained LipiDetective transformer model weights (.pth file)
- YAML configuration file specifying paths to HDF5 prediction files and model parameters

## Outputs

- Spectrum embeddings from the transformer's internal representation layer
- Attention weights for each MS2 spectrum
- Predicted molecular lipid species in shorthand nomenclature (including end-of-sequence token for blanks)

## How to apply

Preprocess raw spectra first using the LipiDetective pipeline: smoothing, baseline correction and peak picking on profile spectra, base peak normalization, trimming MS2 spectra to an m/z range of 50–1600, and storage in an HDF5 file (h5py). Load the trained LipiDetective transformer weights (a .pth file implemented by modifying the PyTorch transformer module, trained with PyTorch Lightning) and run the model in inference mode over the HDF5 spectra. Extract the model's internal representations — spectrum embeddings, attention weights, and the predicted lipid species in shorthand nomenclature — for each MS2 spectrum. These embeddings/attention outputs are the substrate for downstream interpretability work such as integrated gradients attribution computed with the Captum library, which requires access to the embedding layer's gradients. Validate by confirming that predictions match the model's expected output format (lipid species shorthand notation, with end-of-sequence token for blank spectra containing no target lipid).

## Related tools

- **LipiDetective** (Transformer model whose spectrum embeddings and attention weights are extracted during inference over HDF5 MS2 lipid spectra) — github.com/LipiTUM/lipidetective
- **PyTorch** (Framework in which LipiDetective's transformer module was implemented and from which inference is run)
- **h5py** (Library used to load preprocessed MS2 spectra from HDF5 files for model input)
- **Captum** (Library through which integrated gradients gradients over the extracted embeddings are computed for downstream interpretability)
- **pyOpenMS** (Library used for preprocessing raw spectra (smoothing, baseline correction, peak picking) into model-ready form)

## Examples

```
uv run lipidetective --config config/predict.yaml
```

## Evaluation signals

- Output embeddings have a fixed dimensional shape consistent with the transformer's hidden size for every input MS2 spectrum in the HDF5 file
- Attention weights per spectrum are square matrices of size equal to the number of peak tokens, with rows summing to 1 per attention head
- Predictions are lipid species in shorthand nomenclature; blank spectra with no target lipid should yield the end-of-sequence token rather than a lipid name
- Every predicted lipid can be traced back to attention over specific peak tokens, enabling a sanity check that high-attention m/z values fall in the 50–1600 range used during preprocessing

## Limitations

- The model was trained on a relatively small dataset of 268,720 spectra, which may limit embedding quality for lipid species or experimental setups far outside the training distribution
- Peaks relevant for prediction (e.g. 224.0 m/z) lack annotation in the Alex123 database, so not all high-attention peaks can be mapped to known fragments
- It is often not possible to match identifications back to exact spectra because spectrum IDs are rarely provided and metadata may differ due to preprocessing — reproducibility of embedding-to-spectrum mapping depends on retaining spectrum IDs
- Blank MS2 spectra were included so the model learns to predict nothing (end-of-sequence token); spectra unlike these training conditions may yield unreliable embeddings

## Evidence

- [methods] HDF5 storage of preprocessed spectra: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [methods] Spectral preprocessing to m/z 50–1600: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] PyTorch transformer implementation: "LipiDetective was implemented by modifying the transformer module provided by PyTorch"
- [methods] Captum for integrated gradients: "the integrated gradients [30] method was employed as provided by the Captum [31] library"
- [abstract] Transformer identifies lipid species from MS2 spectra: "a transformer model called LipiDetective was developed and trained on a dataset composed of reference spectra measured from lipid standards"
- [readme] LipiDetective supports prediction using mzML files: [evidence span withheld — anchor status: external_doc]
