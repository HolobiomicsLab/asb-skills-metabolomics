---
name: protein-metabolite-integrative-embedding-analysis
description: Use when you have a segmented single-cell feature matrix with per-cell
  protein marker intensities and metabolite intensities of very different dimensionality/variability
  (e.g., ~25 protein vs ~200 metabolic markers) and need a joint latent representation.
when_to_use_negative:
- The data is already a joint embedding or an integrated multi-omics feature table
  — this skill trains the embedding from raw segmented intensity blocks
- Protein and metabolite measurements do not come from the same registered single
  cells (e.g., cross-modality registration via Histone 3/Intercalator and Phosphate
  79 m/z channels was not performed), since branch-wise training assumes paired per-cell
  features
- The analysis goal is integration across cells/datasets of a single modality — multi-omics
  integrators like MNN, Scanorama, Conos, MARIO, or Seurat are the article's stated
  alternatives for that setting
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: scSpaMet
  role: Framework providing the two-branch Keras VAE (AEp protein encoder, AEm metabolite
    encoder) used to compute the joint protein-metabolite latent embedding
  repo: github.com/coskunlab/ScSpaMet
- name: Keras
  role: Deep learning backend implementing the two-branch VAE architecture and its
    training loop
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:29c06e5b1a74e34a5eaf0e01ff8922ac7a6f2404ab6ebf8ac9b9ae84241ab415
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protein-metabolite-integrative-embedding-analysis

## Summary

Trains a two-branch cross-modality Variational Autoencoder (scSpaMet Keras implementation) that fuses normalized single-cell protein (IMC) and metabolite (TOF-SIMS/3D-SMF) intensity blocks into a shared 16-dimensional joint latent embedding, benchmarking reconstruction MAE against single-modality baselines.

## When to use

Use when you have a segmented single-cell feature matrix with per-cell protein marker intensities and metabolite intensities of very different dimensionality/variability (e.g., ~25 protein vs ~200 metabolic markers) and need a joint latent representation.

## When NOT to use

- The data is already a joint embedding or an integrated multi-omics feature table — this skill trains the embedding from raw segmented intensity blocks
- Protein and metabolite measurements do not come from the same registered single cells (e.g., cross-modality registration via Histone 3/Intercalator and Phosphate 79 m/z channels was not performed), since branch-wise training assumes paired per-cell features
- The analysis goal is integration across cells/datasets of a single modality — multi-omics integrators like MNN, Scanorama, Conos, MARIO, or Seurat are the article's stated alternatives for that setting

## Inputs

- Segmented single-cell feature matrix with per-cell protein marker intensities (IMC) and metabolite intensities (TOF-SIMS/3D-SMF) from the 01-04 notebook pipeline
- Normalized proteomic and metabolomic feature blocks split into the two VAE modality branches

## Outputs

- Shared 16-dimensional joint embedding matrix per single cell (joint_embedding.npy)
- Tabulated joint-vs-single-modality reconstruction MAE values (vae_mae_results.csv)
- Trained Keras VAE weights for the joint two-branch model and the AEp/AEm baselines

## How to apply

Load the segmented per-cell protein and metabolite intensity matrix produced by the 01-04 scSpaMet notebook pipeline (IMC protein and SIMS metabolite extraction), then normalize each modality block separately and split them into the two VAE branches. Build the two-branch VAE in scSpaMet's Keras code: encoder branch AEp for protein features and branch AEm for metabolite features, each contributing a latent vector of size 8 that converges into a shared 16-dimensional joint latent embedding. Train the joint VAE on the single-cell feature matrix until convergence, recording the reconstruction MAE for each modality branch. Train protein-only (AEp) and metabolite-only (AEm) single-modality baseline VAEs under matching settings to obtain reference reconstruction MAE values for assessing whether joint modeling degrades per-modality reconstruction.

## Related tools

- **scSpaMet** (Framework providing the two-branch Keras VAE (AEp protein encoder, AEm metabolite encoder) used to compute the joint protein-metabolite latent embedding) — github.com/coskunlab/ScSpaMet
- **Keras** (Deep learning backend implementing the two-branch VAE architecture and its training loop)

## Examples

```
import numpy as np; joint_embedding = vae_encoder.predict([protein_features, metabolite_features]); np.save('joint_embedding.npy', joint_embedding)
```

## Evaluation signals

- Reconstruction MAE per modality is computed and reported for both the joint two-branch VAE and the single-modality VAE baselines in vae_mae_results.csv, enabling direct comparison
- The saved joint embedding matrix in joint_embedding.npy has exactly 16 latent dimensions per single cell (8 per modality branch)
- The latent dimensionality check: each modality encoder branch contributes a latent vector of size 8 before convergence into the shared embedding
- The input feature matrix matches the outputs of the 01-04 intensity-extraction notebook pipeline (per-cell protein and metabolite intensities), confirming correct upstream preprocessing
- Joint-vs-single-modality MAE values are present for both modalities (protein AEp and metabolite AEm), not just the joint model

## Limitations

- The two modalities differ strongly in variability and feature size (~200 metabolic markers vs ~25 protein markers), which is the core imbalance the two-branch design must handle; naive concatenation would be dominated by the metabolite block
- Reconstruction MAE alone is the acceptance criterion; it does not directly validate biological utility of the embedding (downstream metabolite-state and trajectory analyses address that)
- Quality depends on upstream cross-modality single-cell registration; misregistered cells would corrupt paired protein-metabolite training samples

## Evidence

- [full_text] Build the two-branch VAE in scSpaMet's Keras implementation: encoder branch AEp for protein features and branch AEm for metabolite features, converging into a shared 16-dimensional joint latent embedding: [evidence span withheld — anchor status: not_found]
- [full_text] A cross-modality VAE captured a joint protein-metabolite latent variable at the single-cell level by combining a latent vector of size 8 from each modality (yielding a 16-dim joint embedding).: [evidence span withheld — anchor status: not_found]
- [abstract] Deep learning-based joint embedding revealed unique metabolite states within cell types: "Deep learning-based joint embedding revealed unique metabolite states within cell types"
- [readme] 05 VAE joint embedding of protein and metabolite modalities: "05 VAE joint embedding of protein and metabolite modalities"
- [readme] "scSpaMet" folder contains keras code used for VAE analysis: ""scSpaMet" folder contains keras code used for VAE analysis"
