---
name: clustering-agreement-metrics-computation
description: Use when you must show that a trained model's intermediate representations
  progressively discard batch/hospital information while retaining disease-class information,
  layer by layer.
when_to_use_negative:
- Input is a already-aggregated feature table or peak-picked metabolite annotation
  matrix rather than raw/spectral data flowing through a neural network — the layer-scan
  design assumes hidden-layer activations.
- You need batch correction of the data itself (e.g., an output transformed dataset);
  this skill only quantifies and visualizes batch-effect removal inside the model's
  representations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Supplies the trained DenseNet121 model, the run_feature forward-pass step,
    and the .npy demo data whose hidden-layer outputs are scored with NMI/ARI/ASC/KBET/LISI
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend framework on which the DenseNet121 model runs and whose layers are
    hooked to extract activation matrices
- name: Keras 2.3.1
  role: Keras backend for the DenseNet121 architecture from which intermediate-layer
    outputs are collected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:1eff9baac6e34aa9bc57237de8b052266a62617961c7446abbd977e732107df9
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# clustering-agreement-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies progressive batch-effect removal across a deep network's hidden layers (e.g., DenseNet121 in DeepMSProfiler) by scoring layer-wise PCA embeddings with NMI, ARI, and ASC against class labels and KBET/LISI against batch labels.

## When to use

Use when you must show that a trained model's intermediate representations progressively discard batch/hospital information while retaining disease-class information, layer by layer.

## When NOT to use

- Input is a already-aggregated feature table or peak-picked metabolite annotation matrix rather than raw/spectral data flowing through a neural network — the layer-scan design assumes hidden-layer activations.
- You need batch correction of the data itself (e.g., an output transformed dataset); this skill only quantifies and visualizes batch-effect removal inside the model's representations.

## Inputs

- Trained DeepMSProfiler DenseNet121 model (TensorFlow 2.2.0 / Keras 2.3.1)
- Raw metabolomics data in .npy format (converted automatically from .mzML if needed)
- datalist/label file mapping samples (e.g., C01.npy, H01.npy, N01.npy) to disease labels and train/test splits

## Outputs

- layerwise_batch_metrics.csv with NMI/ARI/ASC/KBET/LISI columns populated for every hidden layer
- layerwise_pca_embeddings.npy with per-layer 2D PCA coordinates
- Per-layer PCA visualization plots reproducing Fig. 3d–e

## How to apply

Load the trained DenseNet121 model (TensorFlow 2.2.0 / Keras 2.3.1, adam optimizer, lr 1e-4, batch size 8, ensemble of 10 runs) with the demo .npy metabolomics data and the datalist/label file. Register forward hooks on intermediate layers and run a forward pass (analogous to the package's run_feature step) to collect each layer's activation matrix for all samples. For every hidden layer, compute a PCA projection of the embeddings to a 2D space, then score it with NMI, ARI, and ASC against disease-class labels and with KBET and LISI against batch labels, replicating the layer-by-layer evaluation of Fig. 3d–e. The validation criterion is the metric contrast: as depth increases, class-label agreement (NMI/ARI/ASC) rises while batch-label agreement (KBET/LISI) falls, demonstrating batch-effect removal in the forward pass.

## Related tools

- **DeepMSProfiler** (Supplies the trained DenseNet121 model, the run_feature forward-pass step, and the .npy demo data whose hidden-layer outputs are scored with NMI/ARI/ASC/KBET/LISI) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend framework on which the DenseNet121 model runs and whose layers are hooked to extract activation matrices)
- **Keras 2.3.1** (Keras backend for the DenseNet121 architecture from which intermediate-layer outputs are collected)

## Examples

```
from DeepMSProfiler import *; run_feature(job_dir='DeepMSProfiler/example/out/jobs007')
```

## Evaluation signals

- layerwise_batch_metrics.csv exists with all five metric columns (NMI, ARI, ASC, KBET, LISI) populated for every hidden layer
- Class-label metrics (NMI/ARI/ASC) show a monotonically increasing trend in deeper layers, matching the reported Fig. 3d–e trend
- Batch-label metrics (KBET/LISI) show decreasing relevance to batch labels as layers approach the output
- Per-layer PCA coordinates in layerwise_pca_embeddings.npy form a 2D embedding per hidden layer for all samples
- The same model/ensemble configuration (DenseNet121, adam, lr 1e-4, batch size 8, 10 runs) is used so scores are reproducible

## Limitations

- Requires access to the trained model and per-layer activations; pre-trained serum model (859 samples: 210 healthy, 323 lung nodules, 326 lung cancer) access requires contacting the author
- Environment is pinned to TensorFlow 2.2.0 / Keras 2.3.1 / CUDA 10.1 (CPU-only also supported); newer frameworks may not expose equivalent intermediate-layer hooks
- The demonstrated contrast was validated on the article's lung-cancer / colon-cancer (MetaboLights MTBLS1129) / CCLE metabolomic datasets; other data modalities are untested in this context

## Evidence
