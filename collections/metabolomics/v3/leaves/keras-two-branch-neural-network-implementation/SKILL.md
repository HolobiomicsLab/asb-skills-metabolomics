---
name: keras-two-branch-neural-network-implementation
description: Use when the input is a segmented single-cell feature matrix containing
  matched per-cell protein marker intensities and metabolite intensities, and the
  goal is a joint cross-modality latent embedding or comparison of joint vs. single-modality
  reconstruction quality.
when_to_use_negative:
- Only one modality is present (protein-only or metabolite-only data) — use the single-modality
  VAE baseline instead, as the cross-modality branch requires matched profiles from
  the same single cell
- Input is still raw IMC/SIMS images rather than a segmented per-cell feature table
  — the 01-04 segmentation and intensity-extraction notebooks must run first
- Modality blocks are not aligned per single cell (no cross-modality registration)
  — the joint embedding is meaningless for unmatched protein-metabolite profiles
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scSpaMet
  role: Provides the Keras VAE implementation (encoder branches AEp and AEm) used
    to build and train the joint two-branch protein-metabolite embedding
  repo: github.com/coskunlab/ScSpaMet
- name: Keras
  role: Deep learning framework in which the two-branch VAE encoders and the 16-dim
    joint latent layer are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:8989eee715a5e9097a8f93f6032989a2ccae40db3a0ce733c7a341fd619d6999
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# keras-two-branch-neural-network-implementation

## Summary

A two-branch Keras VAE (encoder branches AEp for protein, AEm for metabolite) that fuses single-cell protein and metabolite profiles into a shared 16-dimensional joint latent embedding (8-dim per modality), as implemented in scSpaMet for joint protein-metabolite single-cell analysis.

## When to use

Use when the input is a segmented single-cell feature matrix containing matched per-cell protein marker intensities and metabolite intensities, and the goal is a joint cross-modality latent embedding or comparison of joint vs. single-modality reconstruction quality.

## When NOT to use

- Only one modality is present (protein-only or metabolite-only data) — use the single-modality VAE baseline instead, as the cross-modality branch requires matched profiles from the same single cell
- Input is still raw IMC/SIMS images rather than a segmented per-cell feature table — the 01-04 segmentation and intensity-extraction notebooks must run first
- Modality blocks are not aligned per single cell (no cross-modality registration) — the joint embedding is meaningless for unmatched protein-metabolite profiles

## Inputs

- Segmented single-cell feature matrix with per-cell protein marker intensities and metabolite intensities from the intensity-extraction notebook (01-04 pipeline outputs)
- Normalized proteomic feature block and metabolomic feature block (two modality branches)
- Single-modality baselines: protein-only AEp and metabolite-only AEm feature blocks

## Outputs

- joint_embedding.npy: shared 16-dimensional (8+8) joint protein-metabolite latent embedding matrix, one row per single cell
- vae_mae_results.csv: reconstruction MAE values for the joint two-branch VAE and the single-modality VAE baselines per modality

## How to apply

Load the per-cell protein and metabolite feature matrix produced by the 01-04 intensity-extraction pipeline, then normalize each modality block and route them into the two encoder branches of the VAE (AEp for proteins, AEm for metabolites) built with scSpaMet's Keras code in the 'scSpaMet' source folder. Each branch compresses to a latent vector of size 8, and the two latents are concatenated into a shared 16-dimensional joint embedding per single cell. Train the joint two-branch VAE until convergence, recording the reconstruction MAE for each modality branch; in parallel train protein-only (AEp) and metabolite-only (AEm) single-modality VAE baselines under matching settings as references. The rationale is that metabolomic and proteomic single-cell data have imbalanced variability and feature size (>200 metabolic markers vs. 25 protein markers), so a shared latent space lets deep-learning-based joint embedding reveal unique metabolite states within cell types. Save the per-cell 16-dim joint embedding to joint_embedding.npy and tabulate joint-vs-baseline MAE in vae_mae_results.csv (notebook 05 in the README).

## Related tools

- **scSpaMet** (Provides the Keras VAE implementation (encoder branches AEp and AEm) used to build and train the joint two-branch protein-metabolite embedding) — github.com/coskunlab/ScSpaMet
- **Keras** (Deep learning framework in which the two-branch VAE encoders and the 16-dim joint latent layer are implemented)

## Examples

```
np.save("joint_embedding.npy", joint_embedding);  # 16-dim per-cell latent from the two-branch Keras VAE in notebook 05
```

## Evaluation signals

- The joint embedding matrix saved to joint_embedding.npy has exactly 16 dimensions per cell (8-dim protein latent + 8-dim metabolite latent) and one row per segmented single cell
- Reconstruction MAE is computed and reported per modality for both the joint two-branch VAE and the single-modality AEp/AEm baselines in vae_mae_results.csv
- The trained two-branch VAE reconstructs each modality with MAE comparable to (or assessed against) the matching single-modality baseline trained under identical settings
- The downstream joint embedding reveals distinct metabolite states within cell types, consistent with the paper's deep-learning joint-embedding finding
- Both modality feature blocks are normalized before being split into the AEp and AEm branches (guarding against the imbalance in variability and feature size between modalities)

## Limitations

- The two modalities are imbalanced in variability and feature size (more than 200 metabolic markers vs. 25 protein markers per cell), which must be handled by normalization and matched latent sizing
- Metabolite coverage from TOF-SIMS is limited by challenges in metabolite annotation for lipids, amino acids, and metabolic pathways, constraining the metabolite branch's feature space
- Relatively small imaging regions from tissue samples (a trade-off between imaging speed and spatial resolution in TOF-SIMS) limit the number of cells feeding the VAE

## Evidence

- [abstract] Deep learning-based joint embedding revealed unique metabolite states within cell types: "Deep learning-based joint embedding revealed unique metabolite states within cell types"
- [results] A cross-modality VAE captured a joint protein-metabolite latent variable at the single-cell level by combining a latent vector of size 8 from each modality (yielding a 16-dim joint embedding).: [evidence span withheld — anchor status: not_found]
- [intro] scSpaMet enables the correlation of more than 200 metabolic markers and 25 protein markers in individual cells within native tissues.: "scSpaMet enables the correlation of more than 200 metabolic markers and 25 protein markers in individual cells within native tissues."
- [readme] 05 VAE joint embedding of protein and metabolite modalities: "05 VAE joint embedding of protein and metabolite modalities"
- [readme] scSpaMet folder contains keras code used for VAE analysis: ""scSpaMet" folder contains keras code used for VAE analysis"
