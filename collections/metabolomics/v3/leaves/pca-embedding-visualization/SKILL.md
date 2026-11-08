---
name: pca-embedding-visualization
description: Use when a trained deep metabolomics model (e.g. DenseNet121 on raw LC-MS
  .npy spectra) must be inspected layer-by-layer to show how hidden representations
  shed batch information and gain disease-class information.
when_to_use_negative:
- Input is a precomputed peak/feature table rather than raw LC-MS spectra processed
  end-to-end by DeepMSProfiler's DenseNet121 — there are no hidden-layer activations
  to extract
- No trained model checkpoint is available and training must not be run (pre-trained
  859-sample model access requires contacting the author)
- Only the final prediction outputs (AUC curves, confusion matrix, RISE heatmaps)
  are needed rather than internal representation analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: DeepMSProfiler
  role: Provides the trained DenseNet121 model, run_feature-style forward hooks, and
    the demo .npy data pipeline whose hidden layers are PCA-projected and scored
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow 2.2.0
  role: Backend in which the DenseNet121 graph is executed and intermediate layer
    activations are captured during forward propagation
- name: Keras 2.3.1
  role: Model API used to register hooks on intermediate DenseNet121 layers and collect
    activation matrices per sample
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:1d09bed786589e7b751a700cbd0836866777be11f96d594eaf30a4eb808169ce
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pca-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts hidden-layer activation matrices from DeepMSProfiler's trained DenseNet121 during forward propagation and projects each layer's embeddings with PCA for 2D visualization, scoring them with NMI/ARI/ASC against disease-class labels and KBET/LISI against batch labels to quantify progressive batch-effect removal.

## When to use

Use when a trained deep metabolomics model (e.g. DenseNet121 on raw LC-MS .npy spectra) must be inspected layer-by-layer to show how hidden representations shed batch information and gain disease-class information.

## When NOT to use

- Input is a precomputed peak/feature table rather than raw LC-MS spectra processed end-to-end by DeepMSProfiler's DenseNet121 — there are no hidden-layer activations to extract
- No trained model checkpoint is available and training must not be run (pre-trained 859-sample model access requires contacting the author)
- Only the final prediction outputs (AUC curves, confusion matrix, RISE heatmaps) are needed rather than internal representation analysis

## Inputs

- Raw metabolomics spectra in .npy format (demo data from the example dir or Baidu Netdisk)
- Datalist/label file mapping FilePath to disease Label and train/test split (e.g. example/datalist.txt)
- Trained DenseNet121 model checkpoint(s) from the 10-run ensemble (job_dir under ../jobs, e.g. jobs007)

## Outputs

- layerwise_batch_metrics.csv with NMI/ARI/ASC/KBET/LISI columns populated for every hidden layer x label-type
- layerwise_pca_embeddings.npy holding per-layer 2D PCA coordinates for all samples
- Per-layer PCA scatter plots colored by batch and by disease class (analogous to Fig. 3d-e)

## How to apply

Load the demo .npy metabolomics data with the datalist/label file and the trained DenseNet121 model (TensorFlow 2.2.0 / Keras 2.3.1, adam optimizer, lr 1e-4, batch size 8, 10-run ensemble). Register forward hooks on intermediate DenseNet121 layers and run the forward pass (analogous to the package's run_feature step) to collect each layer's activation matrix for all samples. For every hidden layer compute a PCA projection to a 2D space and save the coordinates; then score the embeddings with NMI, ARI and ASC against the disease-class labels (cancer/nodule/health) and with KBET and LISI against batch labels, replicating the layer-by-layer evaluation of Fig. 3d–e. Write all metric x layer x label-type scores to layerwise_batch_metrics.csv and the per-layer PCA coordinates to layerwise_pca_embeddings.npy alongside the plots.

## Related tools

- **DeepMSProfiler** (Provides the trained DenseNet121 model, run_feature-style forward hooks, and the demo .npy data pipeline whose hidden layers are PCA-projected and scored) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow 2.2.0** (Backend in which the DenseNet121 graph is executed and intermediate layer activations are captured during forward propagation)
- **Keras 2.3.1** (Model API used to register hooks on intermediate DenseNet121 layers and collect activation matrices per sample)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature
```

## Evaluation signals

- layerwise_batch_metrics.csv exists with NMI/ARI/ASC/KBET/LISI populated for every hidden layer, not just the final layer
- Class-label agreement (NMI/ARI/ASC vs disease labels) rises monotonically or near-monotonically in deeper layers
- Batch-label agreement (KBET/LISI vs batch labels) falls toward the output layer, reproducing the reported trend of Fig. 3d-e
- layerwise_pca_embeddings.npy contains a 2D coordinate array per hidden layer covering all samples in the datalist
- PCA scatter plots show progressive separation of cancer/nodule/health classes and mixing of batches as depth increases

## Limitations

- Requires a trained DenseNet121 checkpoint; the distributed pre-trained serum model (859 samples: 210 healthy, 323 lung nodules, 326 lung cancer) is academic-use only and requires contacting the author
- Environment is pinned to TensorFlow 2.2.0, Keras 2.3.1, Python >= 3.6 (CUDA 10.1 tested; CPU-only also supported), so hooking code must match this stack
- Demo heatmap/feature outputs such as ensemble_RISE.npy are tied to a specific job directory (jobs007) and the 10-run ensemble; results vary with ensemble runs

## Evidence

- [results] Hidden-layer outputs are extracted during forward propagation, PCA-projected for visualisation, and scored with NMI, ARI, and ASC (plus KBET and LISI) against batch and class labels: [evidence span withheld — anchor status: not_found]
