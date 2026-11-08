---
name: hidden-layer-activation-extraction
description: Use when you need to quantify how a trained deep classifier (e.g., DenseNet121
  on raw LC-MS .npy spectra) progressively removes batch-related information layer-by-layer,
  and to visualise hidden-layer embeddings.
when_to_use_negative:
- The model has not yet been trained — you need a fitted DenseNet121 checkpoint (e.g.,
  from run_train) before hidden layers carry meaningful batch/class signal
- You only need final predictions or AUC/confusion-matrix outputs (run_predict with
  plot_auc/plot_cm), not layer-wise representational analysis
- Input data is already a peak-aligned feature table rather than raw spectra — the
  layer-scan interprets DenseNet121's raw-signal representations, not externally engineered
  features
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Provides the trained DenseNet121 model, demo .npy data, datalist/label files,
    and the run_feature forward-pass infrastructure from which hidden-layer activations
    are extracted
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend whose layer-hook mechanism collects intermediate activation matrices
    during the forward pass
- name: Keras 2.3.1
  role: Model-definition API for DenseNet121 whose intermediate layers are hooked
    for activation extraction
- name: Python (>= 3.6)
  role: Implementation language for the hook registration, PCA projection, and NMI/ARI/ASC/KBET/LISI
    scoring scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:a5583d77a9cf003eeb348b18eadeb1a5421c1ea3b165297534442b5af1e87834
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hidden-layer-activation-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Registers forward hooks on intermediate layers of a trained DenseNet121 (DeepMSProfiler, TensorFlow 2.2.0/Keras 2.3.1) to collect activation matrices for every sample during a forward pass, then PCA-projects each layer's embeddings and scores them with NMI/ARI/ASC (class labels) and KBET/LISI (batch labels) to quantify progressive batch-effect removal across network depth.

## When to use

Use when you need to quantify how a trained deep classifier (e.g., DenseNet121 on raw LC-MS .npy spectra) progressively removes batch-related information layer-by-layer, and to visualise hidden-layer embeddings.

## When NOT to use

- The model has not yet been trained — you need a fitted DenseNet121 checkpoint (e.g., from run_train) before hidden layers carry meaningful batch/class signal
- You only need final predictions or AUC/confusion-matrix outputs (run_predict with plot_auc/plot_cm), not layer-wise representational analysis
- Input data is already a peak-aligned feature table rather than raw spectra — the layer-scan interprets DenseNet121's raw-signal representations, not externally engineered features

## Inputs

- Raw metabolomics data in .npy format (demo data from the DeepMSProfiler example dir or Baidu Netdisk; .mzML files can be auto-converted to .npy)
- Datalist/label file mapping sample files to disease-class and train/test split (e.g., DeepMSProfiler/example/datalist.txt)
- Trained DenseNet121 model checkpoint (TensorFlow 2.2.0 / Keras 2.3.1, trained with adam, lr 1e-4, batch size 8, 10 ensemble runs)
- Batch-label annotations per sample (e.g., source hospital) and disease-class labels

## Outputs

- Per-layer activation matrices (layer x samples embeddings collected via forward hooks)
- layerwise_batch_metrics.csv with NMI/ARI/ASC/KBET/LISI columns populated for every hidden layer
- layerwise_pca_embeddings.npy with 2D PCA coordinates per hidden layer
- Per-layer PCA visualization plots (replicating Fig. 3d–e layer scan)

## How to apply

Load the demo .npy metabolomics data with its datalist/label file and the trained DenseNet121 model (adam optimizer, lr 1e-4, batch size 8, ensemble of 10 runs, TF 2.2.0/Keras 2.3.1 backend). Register forward hooks on the intermediate DenseNet121 layers and run the forward pass — analogous to DeepMSProfiler's run_feature step — collecting each layer's activation matrix for all samples. For every hidden layer, compute a PCA projection of the extracted embeddings to a 2D visualization space, then score the space with NMI, ARI and ASC against disease-class labels and with KBET and LISI against batch labels, replicating the layer-by-layer evaluation of Fig. 3d–e. Write all scores (metric x layer x label-type) to layerwise_batch_metrics.csv and save per-layer PCA coordinates as layerwise_pca_embeddings.npy alongside plots. Validate by confirming the class-vs-batch metric contrast reproduces the reported trend: class-label agreement rises while batch-label agreement falls in deeper layers, approaching the output layer.

## Related tools

- **DeepMSProfiler** (Provides the trained DenseNet121 model, demo .npy data, datalist/label files, and the run_feature forward-pass infrastructure from which hidden-layer activations are extracted) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend whose layer-hook mechanism collects intermediate activation matrices during the forward pass)
- **Keras 2.3.1** (Model-definition API for DenseNet121 whose intermediate layers are hooked for activation extraction)
- **Python (>= 3.6)** (Implementation language for the hook registration, PCA projection, and NMI/ARI/ASC/KBET/LISI scoring scripts)

## Examples

```
run_feature(job_dir='DeepMSProfiler/example/out/jobs007')
```

## Evaluation signals

- layerwise_batch_metrics.csv exists and has NMI/ARI/ASC/KBET/LISI values populated for every hidden layer
- Class-label metrics (NMI/ARI/ASC) monotonically increase toward deeper layers, reproducing the reported trend
- Batch-label metrics (KBET/LISI) decrease toward the output layer, demonstrating progressive batch-effect removal
- Per-layer PCA coordinates (layerwise_pca_embeddings.npy) match the number of hidden layers and sample count in the datalist
- 2D PCA visualisations show batch-mixed clusters early in the network separating into disease-class clusters near the output

## Limitations

- Environment is pinned: TensorFlow == 2.2.0, Keras == 2.3.1, Python >= 3.6, CUDA 10.1 (though CPU-only environments are compatible)
- The pre-trained 859-sample serum model (210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author for access
- Batch-label scoring (KBET/LISI) requires explicit batch annotations per sample; without them only class-label metrics can be computed

## Evidence

- [methods] Register forward hooks on intermediate DenseNet121 layers and run the forward pass to collect each layer's activation matrix for all samples: [evidence span withheld — anchor status: not_found]
- [results] Approaching the output layer, relevance to batch labels decreases while relevance to class labels increases: [evidence span withheld — anchor status: not_found]
