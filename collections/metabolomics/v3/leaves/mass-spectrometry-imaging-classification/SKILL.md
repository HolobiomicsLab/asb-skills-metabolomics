---
name: mass-spectrometry-imaging-classification
description: Use when raw MALDI MSI spectra (e.g. imzML/HDF5 with tens of thousands
  of m/z variables) must be classified into tissue classes (normal vs tumor) with
  per-pixel probabilities, and the probabilistic outputs must be mapped back into
  image space to localize classification uncertainty such as at tumor.
when_to_use_negative:
- When a feature table or peak-picked spectra are already available and a classical
  ML method (e.g. SVM) with prior spectral preprocessing is preferred — massNet is
  designed to classify directly from raw spectra without prior preprocessing or peak
  picking
- When the task requires multi-class tissue classification — the paper demonstrates
  only binary class classification (normal vs tumor) and leaves multi-class extension
  for future development
- When reliable ground truth annotation is unavailable — the paper notes annotating
  MSI data from histology alone is an approximation, especially with intra-tumor heterogeneity
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning model providing probabilistic pixel-based classification directly
    from mass spectral data; used to obtain per-spectrum probabilities and reconstruct
    uncertainty maps
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Implementation language for the massNet architecture and spatial reconstruction
    of predictions into image space
- name: Keras
  role: Deep learning platform in which the massNet architecture is implemented
- name: tensorflow
  role: Backend for the Keras-implemented massNet model
- name: SCiLS lab 2020a
  role: Source of the imzML export that is converted to HDF5 for deep learning analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:115c2837ad61b0dc2192c470ebe620fcf281784b954cb6d2da88fe2ae3aa1767
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
  - SCiLS lab 2020a
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectrometry-imaging-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A deep-learning skill for probabilistic pixel-based classification of MALDI mass spectrometry imaging data directly from raw mass spectra, without prior preprocessing or peak picking, and for spatially reconstructing predicted class probabilities and uncertainty to visualize tumor margins.

## When to use

Use when raw MALDI MSI spectra (e.g. imzML/HDF5 with tens of thousands of m/z variables) must be classified into tissue classes (normal vs tumor) with per-pixel probabilities, and the probabilistic outputs must be mapped back into image space to localize classification uncertainty such as at tumor margins.

## When NOT to use

- When a feature table or peak-picked spectra are already available and a classical ML method (e.g. SVM) with prior spectral preprocessing is preferred — massNet is designed to classify directly from raw spectra without prior preprocessing or peak picking
- When the task requires multi-class tissue classification — the paper demonstrates only binary class classification (normal vs tumor) and leaves multi-class extension for future development
- When reliable ground truth annotation is unavailable — the paper notes annotating MSI data from histology alone is an approximation, especially with intra-tumor heterogeneity

## Inputs

- HDF5-formatted MALDI FT-ICR MSI test data from intracranial GBM PDX models (85,062 m/z variables)
- Trained massNet model (Keras/Tensorflow weights)
- imzML files exported from SCiLS lab 2020a (converted to HDF5 before classification)
- Optional: pathologist-annotated H&E regions of interest transferred to the MSI data for ground truth

## Outputs

- Per-spectrum probabilistic class scores (0 to 1) for normal vs tumor
- Spatially reconstructed predicted class label maps in image space
- Per-pixel classification uncertainty maps highlighting elevated uncertainty at tumor margins

## How to apply

Export MSI data from SCiLS lab 2020a as imzML and convert to HDF5 for deep learning analysis. Load the trained massNet model (implemented in Python 3.6.12 with Keras 2.2.0 on a Tensorflow 1.8.0 backend) and apply it to the HDF5 MALDI FT-ICR MSI test data (85,062 m/z variables) to obtain a per-spectrum probabilistic estimate (0 to 1) of the predicted class label for normal and tumor. Spatially reconstruct the predicted class labels and their probabilities back into image space to reveal spatial patterns per class, then compute per-pixel classification uncertainty from the probabilistic outputs and map it back into image space. Validation: inspect the spatially reconstructed uncertainty maps and confirm that uncertainty is elevated at the tumor margin relative to tumor core and normal tissue, consistent with an infiltrative edge containing mixed normal and tumor cells.

## Related tools

- **massNet** (Deep learning model providing probabilistic pixel-based classification directly from mass spectral data; used to obtain per-spectrum probabilities and reconstruct uncertainty maps) — https://github.com/wabdelmoula/massNet
- **Python** (Implementation language for the massNet architecture and spatial reconstruction of predictions into image space)
- **Keras** (Deep learning platform in which the massNet architecture is implemented)
- **tensorflow** (Backend for the Keras-implemented massNet model)
- **SCiLS lab 2020a** (Source of the imzML export that is converted to HDF5 for deep learning analysis)

## Examples

```
git clone https://github.com/wabdelmoula/massNet; python massNet_classify.py --input msi_test.h5 --model massNet_weights.h5  # per-spectrum probabilities (0-1) for normal vs tumor
```

## Evaluation signals

- Spatially reconstructed uncertainty maps must show higher classification uncertainty at tumor margins than in tumor core or normal tissue
- Per-spectrum probabilities must fall in the 0 to 1 range for the predicted class label
- Classification accuracy on the test set (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables) should be evaluated with ROC, accuracy, recall, precision, and F1-score and should exceed the SVM baseline
- Model stability should be confirmed via 5-fold cross validation
- Spatial maps must be consistent with pathologist-annotated H&E regions transferred to the MSI data

## Limitations

- Only binary class classification (normal vs tumor) is demonstrated; multi-class classification is left for future development
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity
- Tumor margin ground truth annotation is challenging and requires more pathological investigation
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text (README provides Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0)

## Evidence
