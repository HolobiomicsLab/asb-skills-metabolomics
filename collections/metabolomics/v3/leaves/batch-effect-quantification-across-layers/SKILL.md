---
name: batch-effect-quantification-across-layers
description: Use when a trained DenseNet121 (DeepMSProfiler ensemble) is available
  on raw LC-MS .npy data and you need to measure, layer by layer, whether hidden representations
  shed batch/hospital variation while retaining disease-class information.
when_to_use_negative:
- The model is untrained or only a feature/peak table is available — the layer-scan
  requires a trained DenseNet121's hidden activations, not a precomputed feature matrix
- Samples lack batch/hospital labels — KBET and LISI against batch labels cannot be
  computed, so batch-effect removal cannot be quantified
- The goal is metabolite-level biomarker interpretation rather than representation
  auditing — use the RISE ensemble feature heatmaps (run_feature/show_feature) instead
tools:
- name: DeepMSProfiler
  role: Provides the trained DenseNet121 ensemble and the run_feature step whose forward
    pass yields the hidden-layer activations scored here
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend for registering forward hooks on intermediate DenseNet121 layers and
    collecting activation matrices
- name: Keras 2.3.1
  role: Model API used to access intermediate DenseNet121 layer outputs during forward
    propagation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:25af20abd1073491948e60e48a1d11136fb84df4c7e02745da18d205577e25d7
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
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# batch-effect-quantification-across-layers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies progressive batch-effect removal inside DeepMSProfiler's DenseNet121 by scoring each hidden layer's activations with NMI/ARI/ASC against class labels and KBET/LISI against batch labels, plus PCA projection per layer, replicating the layer-scan of Fig. 3d-e.

## When to use

Use when a trained DenseNet121 (DeepMSProfiler ensemble) is available on raw LC-MS .npy data and you need to measure, layer by layer, whether hidden representations shed batch/hospital variation while retaining disease-class information.

## When NOT to use

- The model is untrained or only a feature/peak table is available — the layer-scan requires a trained DenseNet121's hidden activations, not a precomputed feature matrix
- Samples lack batch/hospital labels — KBET and LISI against batch labels cannot be computed, so batch-effect removal cannot be quantified
- The goal is metabolite-level biomarker interpretation rather than representation auditing — use the RISE ensemble feature heatmaps (run_feature/show_feature) instead

## Inputs

- Raw metabolomics .npy spectral files (or .mzML auto-converted to .npy) from multiple batches/hospitals
- Datalist/label file mapping FilePath, Label and Dataset (train/test) per sample, e.g. example/datalist.txt
- Trained DenseNet121 model from the DeepMSProfiler ensemble (adam, lr 1e-4, batch size 8, 10 runs)
- Batch/hospital label vector for every sample (for KBET and LISI)

## Outputs

- layerwise_batch_metrics.csv with NMI/ARI/ASC/KBET/LISI columns populated for every hidden layer
- Per-layer PCA 2D coordinates saved as layerwise_pca_embeddings.npy
- Per-layer PCA visualization plots showing the class-vs-batch contrast across depth

## How to apply

Load the demo .npy metabolomics data, the datalist/label file, and the trained DenseNet121 model (TensorFlow 2.2.0 / Keras 2.3.1, adam, lr 1e-4, batch size 8, 10 ensemble runs). Register forward hooks on intermediate layers and run the forward pass — analogous to DeepMSProfiler's run_feature step — collecting each layer's activation matrix for all samples. For every hidden layer compute a PCA projection to a 2D visualization space, then score that embedding with NMI, ARI and ASC against disease-class labels and with KBET and LISI against batch labels, replicating the layer-by-layer evaluation of Fig. 3d-e. The expected signature is that, approaching the output layer, agreement with class labels rises while agreement with batch labels falls, evidencing progressive batch-effect removal inside the network rather than a preprocessing step.

## Related tools

- **DeepMSProfiler** (Provides the trained DenseNet121 ensemble and the run_feature step whose forward pass yields the hidden-layer activations scored here) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend for registering forward hooks on intermediate DenseNet121 layers and collecting activation matrices)
- **Keras 2.3.1** (Model API used to access intermediate DenseNet121 layer outputs during forward propagation)

## Examples

```
from DeepMSProfiler import *; run_feature(job_dir='DeepMSProfiler/example/out/jobs007')
```

## Evaluation signals

- layerwise_batch_metrics.csv exists and every hidden layer has NMI, ARI, ASC, KBET and LISI values populated
- Class-label metrics (NMI/ARI/ASC) monotonically or near-monotonically increase toward deeper layers, reproducing Fig. 3d-e
- Batch-label metrics (KBET/LISI) decrease toward deeper layers, showing the class-vs-batch metric contrast
- layerwise_pca_embeddings.npy contains a 2D PCA coordinate array per hidden layer matching the number of samples
- Scoring uses the ensemble-trained model (10 runs, adam, lr 1.0e-04) rather than a single untrained network

## Limitations

- Scores depend on the specific DenseNet121 architecture and training setup (TensorFlow 2.2.0 / Keras 2.3.1); other backbones or versions may change the layer-by-layer trend
- Requires batch labels; without hospital/batch annotations the batch-side metrics cannot be computed
- Access to the pre-trained 859-sample serum model (210 healthy, 323 lung nodules, 326 lung cancer) requires contacting the author, so reproduction may require retraining from the demo data

## Evidence

- [intro] DeepMSProfiler effectively removes undesirable batch effects and variations across different hospitals: "DeepMSProfiler effectively removes undesirable batch effects and variations across different hospitals"
- [abstract] Model flow and ablation experiments demonstrate that DeepMSProfiler overcomes inter-hospital variability and effects of unknown metabolites signals: "Model flow and ablation experiments demonstrate that DeepMSProfiler overcomes inter-hospital variability and effects of unknown metabolites signals"
- [readme] run_feature(job_dir='DeepMSProfiler/example/out/jobs007'): "run_feature(job_dir='DeepMSProfiler/example/out/jobs007')"
- [readme] [INFO] run model with args: DenseNet121 adam lr 1.0e-04 0: "[INFO] run model with args: DenseNet121 adam lr 1.0e-04 0"
