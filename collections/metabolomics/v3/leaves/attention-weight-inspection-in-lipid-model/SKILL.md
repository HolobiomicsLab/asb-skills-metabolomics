---
name: attention-weight-inspection-in-lipid-model
description: Use when you have a trained transformer lipid identification model plus
  preprocessed MS2 spectra and need to verify that the predicted lipid tokens are
  attributable to specific, known fragment m/z peaks.
when_to_use_negative:
- Input is raw profile spectra that have not undergone smoothing, baseline correction,
  peak picking, base peak normalization, and trimming to the 50–1600 m/z range — the
  model expects preprocessed peak lists.
- The goal is to evaluate overall spectral similarity rather than inspect which individual
  peaks drive a token prediction — integrated gradients over embeddings is specific
  to interpretable peak-level attribution.
- No trained transformer model weights are available and training from scratch is
  required; this skill assumes inference and post-hoc attribution on a pre-existing
  checkpoint.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: Captum
  role: computes integrated gradients attributions over the input peak embeddings
    for each predicted lipid token
- name: PyTorch
  role: framework the LipiDetective transformer module is implemented with and used
    to run inference on test spectra
- name: h5py
  role: reads the preprocessed HDF5 spectra files that serve as inference input
- name: LipiDetective
  role: the pre-trained transformer model whose attention weights and predictions
    are inspected
  repo: https://github.com/LipiTUM/lipidetective
- name: Alex123
  role: fragment annotation database used to match top-attributed peaks to known interpretable
    fragments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:64f3467b3bf3a4f312a8c638b1e59b2270f1d92d94fba1c5b73a8c88fa86f814
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - Alex123
  - Captum
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# attention-weight-inspection-in-lipid-model

## Summary

A validation skill for transformer-based lipid identification models: extract the attention weights and compute integrated gradients attributions over input peak embeddings to confirm the model's predictions are driven by humanly interpretable fragment peaks rather than uninterpretable spectral artifacts.

## When to use

Use when you have a trained transformer lipid identification model plus preprocessed MS2 spectra and need to verify that the predicted lipid tokens are attributable to specific, known fragment m/z peaks.

## When NOT to use

- Input is raw profile spectra that have not undergone smoothing, baseline correction, peak picking, base peak normalization, and trimming to the 50–1600 m/z range — the model expects preprocessed peak lists.
- The goal is to evaluate overall spectral similarity rather than inspect which individual peaks drive a token prediction — integrated gradients over embeddings is specific to interpretable peak-level attribution.
- No trained transformer model weights are available and training from scratch is required; this skill assumes inference and post-hoc attribution on a pre-existing checkpoint.

## Inputs

- pre-trained transformer model weights (.pth file)
- preprocessed MS2 spectra in HDF5 format (base-peak normalized, m/z range 50–1600)
- known lipid fragment annotation list (e.g. Alex123)
- lipid shorthand nomenclature labels for predicted tokens

## Outputs

- spectrum embeddings, attention weights, and predicted lipid tokens from inference pass
- per-peak integrated gradients attribution scores
- peak_attributions.csv with columns for m/z, predicted token, and attribution score
- annotated comparison of top peaks vs known fragments

## How to apply

Load the pre-trained LipiDetective transformer weights (.pth) and the preprocessed test spectra stored in HDF5 format. Run the model in inference mode to extract the spectrum embeddings, attention weights, and predicted lipid tokens. Using Captum's integrated gradients implementation, compute per-peak attributions over the input peak embeddings for each predicted lipid token, then aggregate the values into a table keyed by m/z and predicted token. Compare the highest-attribution peaks against known lipid fragment annotations from the Alex123 database — for example, verifying that the 255.2 m/z peak (known 16:0 fatty acid fragment) drives the 16:0 prediction for a PC 16:0_18:1 spectrum. Note peaks such as 224.0 m/z that are highly attributed yet lack Alex123 annotations, as these indicate potentially novel fragments the model exploits.

## Related tools

- **Captum** (computes integrated gradients attributions over the input peak embeddings for each predicted lipid token)
- **PyTorch** (framework the LipiDetective transformer module is implemented with and used to run inference on test spectra)
- **h5py** (reads the preprocessed HDF5 spectra files that serve as inference input)
- **LipiDetective** (the pre-trained transformer model whose attention weights and predictions are inspected) — https://github.com/LipiTUM/lipidetective
- **Alex123** (fragment annotation database used to match top-attributed peaks to known interpretable fragments)

## Examples

```
import torch, captum; from lipietective import model; ig = IntegratedGradients(model); attn = ig.attribute(inputs, target=pred_token_idx)
```

## Evaluation signals

- The top-attributed peak for a PC 16:0_18:1 prediction is 255.2 m/z, matching the known 16:0 fatty acid fragment reported in the paper's integrated gradients analysis.
- peak_attributions.csv exists with columns for m/z, predicted lipid token, and attribution score.
- Attention weights extracted during inference are non-degenerate (not uniform, not zero) across peaks in the spectrum embedding.
- Peaks such as 224.0 m/z that carry high attribution but lack Alex123 annotations are flagged rather than silently dropped.
- Predicted lipid tokens from the inspected inference run match the model's normal output nomenclature (shorthand lipid species).

## Limitations

- Peaks relevant for prediction, such as 224.0 m/z, may lack annotation in the Alex123 database, so not every high-attribution peak can be mapped to a known fragment.
- The validation is qualitative — no exact numeric attribution threshold is imposed beyond consistency with the paper's reported interpretable peaks.
- The skill depends on the model's preprocessed input format: spectra must already be smoothed, baseline-corrected, peak-picked, base-peak normalized, and trimmed to 50–1600 m/z.
- Attribution methods like integrated gradients can highlight peaks the model uses but offer no guarantee that every focused peak has a biologically meaningful fragment identity.

## Evidence

- [abstract] integrated gradients attributions: "Analysis of the integrated gradients reveals that LipiDetective focuses on relevant peaks that can be matched to known fragments and are thus humanly interpretable"
- [results] peak-level attribution and fragment matching: [evidence span withheld — anchor status: not_found]
- [methods] integrated gradients method and library: "the integrated gradients [30] method was employed as provided by the Captum [31] library"
- [results] unannotated relevant peaks: [evidence span withheld — anchor status: not_found]
- [readme] LipiDetective as transformer framework: "**LipiDetective** is a deep learning framework designed for the identification of molecular lipid species from tandem mass spectra."
- [methods] HDF5 input spectra: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
