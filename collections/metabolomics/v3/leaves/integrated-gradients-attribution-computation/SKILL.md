---
name: integrated-gradients-attribution-computation
description: 'Use when a trained transformer model has produced lipid token predictions
  on MS2 spectra and you need per-peak interpretability: attributing each predicted
  lipid token back to specific m/z peaks to compare them against known fragment annotations.'
when_to_use_negative:
- The model under study is not LipiDetective or another transformer over peak embeddings
  — Captum integrated gradients must be applied to a differentiable model with input
  embeddings
- The goal is plain database-based peak matching for lipid identification (use a fragment
  database search, e.g. LipiDex or Alex123) rather than explaining model predictions
- The spectra are raw profile data — preprocessing (smoothing, baseline correction,
  peak picking, normalization) must be completed first so inputs are discrete peaks
  the model consumed
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3391
tools:
- name: Captum
  role: Provides the integrated gradients method used to compute per-peak attributions
    of the input peak embeddings for each predicted lipid token
- name: PyTorch
  role: Runs the modified transformer module underlying LipiDetective in inference
    mode to obtain embeddings, attention weights, and predictions before attribution
- name: LipiDetective
  role: The transformer model whose pre-trained weights and prediction pipeline supply
    the model and spectra being interpreted
  repo: github.com/LipiTUM/lipidetective
- name: h5py
  role: Loads the preprocessed HDF5 test spectra that serve as attribution inputs
- name: Alex123
  role: Reference fragment database used to match top-attributed peaks to known fragments
    and assess human interpretability
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:1500956d837726457ed099f5bccafe02969910e9da975e8af1412a97543a8ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Alex123
  - Captum
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# integrated-gradients-attribution-computation

## Summary

Computes per-peak integrated gradients attributions over the input peak embeddings of the LipiDetective transformer to reveal which MS2 fragment peaks drive each predicted lipid token, demonstrating that predictions rest on humanly interpretable fragments (e.g. the 255.2 m/z 16:0 fatty acid fragment).

## When to use

Use when a trained transformer model has produced lipid token predictions on MS2 spectra and you need per-peak interpretability: attributing each predicted lipid token back to specific m/z peaks to compare them against known fragment annotations.

## When NOT to use

- The model under study is not LipiDetective or another transformer over peak embeddings — Captum integrated gradients must be applied to a differentiable model with input embeddings
- The goal is plain database-based peak matching for lipid identification (use a fragment database search, e.g. LipiDex or Alex123) rather than explaining model predictions
- The spectra are raw profile data — preprocessing (smoothing, baseline correction, peak picking, normalization) must be completed first so inputs are discrete peaks the model consumed

## Inputs

- Pre-trained LipiDetective transformer model weights (saved .pth file)
- Preprocessed MS2 test spectra stored in HDF5 format (smoothed, baseline-corrected, peak-picked, base-peak normalized, trimmed to 50–1600 m/z)
- Model predictions (lipid tokens in shorthand nomenclature) and spectrum embeddings from an inference pass
- Known lipid fragment annotation list for cross-checking (e.g. the Alex123 database)

## Outputs

- Per-peak attribution table keyed by m/z and predicted token (peak_attributions.csv with columns for m/z, predicted token, and attribution score)
- Ranked list of top-attributed peaks per predicted lipid token for comparison against known fragment m/z values

## How to apply

Load the pre-trained LipiDetective transformer weights (.pth) and the preprocessed HDF5 test spectra (peak-picked, base-peak normalized, trimmed to 50–1600 m/z), then run the model in inference mode to extract spectrum embeddings, attention weights, and predictions. Using Captum's IntegratedGradients, compute attributions of the input peak embeddings with respect to each predicted lipid token. Aggregate per-peak values into per-peak attribution tables keyed by m/z and predicted token, then compare the top-attributed peaks against known lipid fragment annotations from the Alex123 database to confirm the model focuses on interpretable peaks. The rationale, per the paper, is to show the model does not match overall spectra but identifies relevant lipid components — e.g. for a PC 16:0_18:1 spectrum the 255.2 m/z peak (the known 16:0 fatty acid fragment) should dominate attribution for the 16:0 token. Note that some relevant peaks (e.g. 224.0 m/z) legitimately lack Alex123 annotation and should be reported as such rather than discarded.

## Related tools

- **Captum** (Provides the integrated gradients method used to compute per-peak attributions of the input peak embeddings for each predicted lipid token)
- **PyTorch** (Runs the modified transformer module underlying LipiDetective in inference mode to obtain embeddings, attention weights, and predictions before attribution)
- **LipiDetective** (The transformer model whose pre-trained weights and prediction pipeline supply the model and spectra being interpreted) — github.com/LipiTUM/lipidetective
- **h5py** (Loads the preprocessed HDF5 test spectra that serve as attribution inputs)
- **Alex123** (Reference fragment database used to match top-attributed peaks to known fragments and assess human interpretability)

## Examples

```
from captum.attr import IntegratedGradients; ig = IntegratedGradients(model); attrs = ig.attribute(peak_embeddings, target=predicted_token_idx)
```

## Evaluation signals

- For a PC 16:0_18:1 spectrum, the peak at 255.2 m/z (the known 16:0 fatty acid fragment) must be the key driver of the attribution for the 16:0 first-fatty-acid token
- Top-attributed peaks qualitatively match known lipid fragment m/z values that the paper reports as humanly interpretable (no exact numeric threshold required)
- peak_attributions.csv contains rows with all three columns: m/z, predicted token, and attribution score
- Known-but-unannotated relevant peaks (e.g. 224.0 m/z) appear among high-attribution peaks and are reported as lacking Alex123 annotation rather than filtered out
- Attributions were computed per predicted lipid token, not as a single pooled spectrum-level score

## Limitations

- Some peaks highly relevant to prediction (e.g. 224.0 m/z) have no annotation in the Alex123 database, so interpretability checks cannot fully classify every top-attributed peak
- Interpretability here is qualitative — the paper reports no exact numeric attribution threshold for confirming a peak is 'the' driver
- Lack of interpretability of DL models in lipidomics is itself flagged as a challenge; attribution results depend on having the correct pre-trained weights and matching preprocessing
- The repo README does not provide a dedicated attribution command — the attribution analysis must be scripted against the LipiDetective model using Captum

## Evidence

- [methods] integrated gradients employed via Captum: "the integrated gradients [30] method was employed as provided by the Captum [31] library"
- [abstract] abstract claim of interpretable peaks: "Analysis of the integrated gradients reveals that LipiDetective focuses on relevant peaks that can be matched to known fragments and are thus humanly interpretable"
- [methods] Alex123 used for fragment matching: "the Alex123 database [19] was used to match peaks in the MS2 spectra to known fragments"
- [results] unannotated relevant peak 224.0 m/z: [evidence span withheld — anchor status: not_found]
- [readme] transformer prediction of lipid species (README): [evidence span withheld — anchor status: external_doc]
