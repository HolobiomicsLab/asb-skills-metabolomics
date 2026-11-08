---
name: spatial-prediction-map-reconstruction
description: Use when you have per-pixel probabilistic class scores from a trained
  classifier applied to mass spectrometry imaging data and need to map predicted labels
  and classification uncertainty back into image space to inspect tumor-margin heterogeneity.
when_to_use_negative:
- When the input data has not yet been classified by massNet and only raw spectra
  are available — run the classifier first to obtain per-spectrum probabilities.
- When the goal is peak identification or m/z annotation rather than spatial visualization
  of predictions — use the msiPL/VAE pipeline instead.
- When the MSI data is not spatially rastered (e.g., single-spectrum or bulk measurements)
  — there is no image space to reconstruct into.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3444
tools:
- name: massNet
  role: Deep learning model that produces per-spectrum probabilistic class scores
    which are then spatially reconstructed into image space
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Framework used to implement the massNet architecture and execute model inference
    on MSI spectra
- name: tensorflow
  role: Backend for Keras model execution during per-spectrum probabilistic prediction
- name: Python
  role: Primary language for implementing spatial reconstruction and uncertainty mapping
    logic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:f23d9949a91815d3e1a41f731ac7529ba6b5f03fa8cc08463cecaaae43026be6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spatial-prediction-map-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs massNet's per-spectrum probabilistic class predictions (normal vs. tumor) back into the original image space of MALDI FT-ICR MSI data, producing spatially resolved prediction maps and per-pixel classification uncertainty maps that reveal elevated uncertainty at the tumor margin.

## When to use

Use when you have per-pixel probabilistic class scores from a trained classifier applied to mass spectrometry imaging data and need to map predicted labels and classification uncertainty back into image space to inspect tumor-margin heterogeneity.

## When NOT to use

- When the input data has not yet been classified by massNet and only raw spectra are available — run the classifier first to obtain per-spectrum probabilities.
- When the goal is peak identification or m/z annotation rather than spatial visualization of predictions — use the msiPL/VAE pipeline instead.
- When the MSI data is not spatially rastered (e.g., single-spectrum or bulk measurements) — there is no image space to reconstruct into.

## Inputs

- HDF5-formatted MALDI FT-ICR MSI dataset with 85,062 m/z variables (converted from imzML exported from SCiLS Lab 2020a)
- Trained massNet model (Keras/TensorFlow, Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0 backend)
- Pathologist-annotated H&E tumor region annotations transferred to MSI pixel coordinates (for validation overlay)

## Outputs

- Spatially reconstructed prediction-class map in original image coordinates
- Per-pixel probabilistic score map (0 to 1) for normal vs. tumor classes
- Per-pixel classification uncertainty map highlighting elevated uncertainty at tumor margins

## How to apply

Load the HDF5-formatted MALDI FT-ICR MSI dataset (e.g., 85,062 m/z variables from intracranial GBM PDX models) and the trained massNet model. Apply massNet to obtain the per-spectrum probabilistic estimate (0 to 1) of the predicted class label for normal and tumor tissue. Spatially reconstruct the predicted class labels and their associated probabilities back into the original image coordinates, preserving pixel positions from the original raster acquisition (100 µm spatial resolution). Compute per-pixel classification uncertainty from the probabilistic outputs and map it back into image space. Inspect the reconstructed maps to confirm that uncertainty is elevated at the tumor margin relative to the tumor core or normal tissue, consistent with infiltrative edges containing mixed normal and tumor cells. Validate the spatial patterns against pathologist-annotated H&E regions transferred onto the MSI data.

## Related tools

- **massNet** (Deep learning model that produces per-spectrum probabilistic class scores which are then spatially reconstructed into image space) — https://github.com/wabdelmoula/massNet
- **Keras** (Framework used to implement the massNet architecture and execute model inference on MSI spectra)
- **tensorflow** (Backend for Keras model execution during per-spectrum probabilistic prediction)
- **Python** (Primary language for implementing spatial reconstruction and uncertainty mapping logic)

## Examples

```
import h5py; import keras; model = keras.models.load_model('massNet_model.h5'); data = h5py.File('msi_data.h5','r'); probs = model.predict(data['spectra']); # reshape probs to spatial raster and map uncertainty per pixel
```

## Evaluation signals

- The spatially reconstructed uncertainty map must show visibly higher classification uncertainty at tumor margins than in tumor core or normal tissue regions.
- Prediction accuracy across most tissue regions should be consistent with massNet's reported higher accuracy compared to support vector machine on the same test set.
- Per-pixel probability values must be bounded between 0 and 1 and sum to 1 across the two classes (normal, tumor).
- Reconstructed maps must preserve spatial registration with the original 100 µm raster grid of the MALDI FT-ICR acquisition.
- Tumor-classified pixels in the reconstructed map should overlap substantially with pathologist-annotated tumor regions transferred from H&E.

## Limitations

- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially in the presence of intra-tumor heterogeneity.
- Tumor margin ground truth annotation would require more pathological investigation to validate uncertainty boundaries.
- The massNet model was demonstrated only for binary class classification; extension to multi-class spatial mapping is left for future development.

## Evidence
