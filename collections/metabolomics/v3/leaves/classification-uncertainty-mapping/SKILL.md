---
name: classification-uncertainty-mapping
description: Use when you have per-spectrum probabilistic outputs (0–1) from a pixel-based
  classifier on mass spectrometry imaging data and need to visualize where classification
  confidence breaks down, e.g. to detect infiltrative tumor margins versus tumor core
  or normal tissue.
when_to_use_negative:
- You only need a single global accuracy/ROC metric for the classifier — this skill
  is specifically about spatially resolved uncertainty, not aggregate performance
  benchmarking (use 5-fold cross-validation and ROC/accuracy/recall/precision/F1 metrics
  instead).
- Input spectra have not yet been classified — you need the trained massNet model's
  probabilistic per-spectrum outputs before uncertainty can be computed.
- The tissue type or tumor lacks a meaningful spatial margin structure, or ground
  truth annotation cannot be established — the article notes ground truth annotation
  based solely on annotated histology is an approximation, especially with intra-tumor
  heterogeneity.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning classifier that produces the per-spectrum probabilistic class
    scores (0–1) from which uncertainty maps are computed
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Platform in which the massNet architecture used for probabilistic prediction
    and uncertainty computation is implemented
- name: tensorflow
  role: Backend platform executing the trained massNet model when applied to the HDF5
    MSI test data
- name: Python
  role: Implementation language for model application, spatial reconstruction, and
    uncertainty mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:6cd8d549e66cec96a7fea79a0e63b86f37fb103a21975d6a6690e69dba2bd62b
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# classification-uncertainty-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Spatially maps massNet's per-spectrum probabilistic class scores (0–1) and derived per-pixel classification uncertainty back into image space to reveal regions of mixed tissue identity, such as infiltrative tumor margins in MALDI FT-ICR MSI data.

## When to use

Use when you have per-spectrum probabilistic outputs (0–1) from a pixel-based classifier on mass spectrometry imaging data and need to visualize where classification confidence breaks down, e.g. to detect infiltrative tumor margins versus tumor core or normal tissue.

## When NOT to use

- You only need a single global accuracy/ROC metric for the classifier — this skill is specifically about spatially resolved uncertainty, not aggregate performance benchmarking (use 5-fold cross-validation and ROC/accuracy/recall/precision/F1 metrics instead).
- Input spectra have not yet been classified — you need the trained massNet model's probabilistic per-spectrum outputs before uncertainty can be computed.
- The tissue type or tumor lacks a meaningful spatial margin structure, or ground truth annotation cannot be established — the article notes ground truth annotation based solely on annotated histology is an approximation, especially with intra-tumor heterogeneity.

## Inputs

- Trained massNet model (Keras/TensorFlow)
- HDF5-formatted MALDI FT-ICR MSI test data with per-pixel mass spectra (e.g. 85,062 m/z variables from four intracranial GBM PDX models)
- Spatial coordinates of each spectrum for image-space reconstruction (from imzML/HDF5 pixel grid)

## Outputs

- Spatially reconstructed predicted-class-label map (normal vs tumor) in image space
- Per-pixel probabilistic score map (0–1) in image space
- Per-pixel classification uncertainty map highlighting tumor-margin regions

## How to apply

Load the trained massNet model and the HDF5-converted MSI test data (e.g. 85,062 m/z variables from four intracranial GBM PDX models) and apply the classifier to obtain a per-spectrum probabilistic estimate (0 to 1) of the predicted class label for normal versus tumor. Spatially reconstruct the predicted labels and their probabilities back into image space to reveal spatial patterns associated with each class (as in Figure 5a-c). Then compute a classification uncertainty value per pixel from the probabilistic outputs and map those uncertainty values back into image space. Inspect the reconstructed uncertainty maps to confirm whether uncertainty is elevated at the tumor margin relative to tumor core or normal tissue — elevated margin uncertainty is the expected signature of an infiltrative edge containing mixed normal and tumor cells. Implementation uses Python with Keras/TensorFlow (README: Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0 backend).

## Related tools

- **massNet** (Deep learning classifier that produces the per-spectrum probabilistic class scores (0–1) from which uncertainty maps are computed) — https://github.com/wabdelmoula/massNet
- **Keras** (Platform in which the massNet architecture used for probabilistic prediction and uncertainty computation is implemented)
- **tensorflow** (Backend platform executing the trained massNet model when applied to the HDF5 MSI test data)
- **Python** (Implementation language for model application, spatial reconstruction, and uncertainty mapping)

## Examples

```
# load HDF5 MSI test data and trained massNet model; predict per-spectrum class probabilities; reshape prob/uncertainty arrays to pixel-grid image dimensions
```

## Evaluation signals

- The spatially reconstructed uncertainty map must show higher classification uncertainty at tumor margins than in tumor core or normal tissue regions.
- Spatially reconstructed predicted-class maps must visually agree with pathologist-annotated tumor regions transferred from consecutive H&E sections.
- Per-spectrum probabilities must lie in the valid range 0 to 1 and map back onto the original pixel grid at the acquisition spatial resolution (100 µm).
- Number of reconstructed pixels must match the number of classified spectra in the test set (e.g. 9,798 normal and 2,861 tumor spectra).

## Limitations

- Ground truth annotation of the MSI data based solely on annotated histology is an approximation that might not be optimal, especially in the case of intra-tumor heterogeneity.
- Tumor-margin ground truth is challenging and would require more pathological investigation, so margin-uncertainty interpretation should be treated as consistent evidence rather than validated ground truth.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article's discussion section (README does supply Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0).
- massNet as published demonstrates only binary (normal vs tumor) classification; multi-class uncertainty mapping is left for future development.

## Evidence

- [results] spatially mapped probabilistic predictions show margin uncertainty: [evidence span withheld — anchor status: not_found]
- [results] infiltrative edge interpretation: [evidence span withheld — anchor status: not_found]
- [results] workflow: reconstruct probabilities in image space: [evidence span withheld — anchor status: not_found]
