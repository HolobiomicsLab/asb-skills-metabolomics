---
name: single-cell-multiomic-joint-embedding
description: Use when a per-cell feature matrix contains paired protein and metabolite
  intensity blocks of unequal feature size and you need a joint low-dimensional embedding
  to compare reconstruction against single-modality VAE baselines.
when_to_use_negative:
- Data is raw IMC/SIMS imagery that has not yet been registered and segmented into
  per-cell intensities — run notebooks 01–04 first
- Only one modality is available — use the single-modality VAE baseline (protein-only
  AEp or metabolite-only AEm) instead of the cross-modality joint model
- Cells lack cross-modality correspondence (no single-cell registration via Histone
  3/Intercalator IMC and Phosphate 79 m/z TOF-SIMS channels), so protein and metabolite
  rows cannot be paired per cell
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: scSpaMet
  role: Provides the Keras VAE implementation ('scSpaMet' source folder) used to build
    and train the two-branch joint protein-metabolite embedding
  repo: github.com/coskunlab/ScSpaMet
- name: Keras
  role: Deep learning framework in which the encoder branches AEp (protein) and AEm
    (metabolite) and the shared 16-dim joint latent embedding are implemented and
    trained
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:d63f459a1a363eda056b11d1c875e4a9c0a2f2272ff038d35fad0ef0edc8034f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-multiomic-joint-embedding

## Summary

Train a two-branch cross-modality Variational Autoencoder (scSpaMet Keras implementation) that fuses segmented single-cell protein (IMC) and metabolite (TOF-SIMS/3D-SMF) intensities into a shared 16-dimensional joint embedding (8-dim latent per modality), enabling joint protein-metabolite analysis of single cells.

## When to use

Use when a per-cell feature matrix contains paired protein and metabolite intensity blocks of unequal feature size and you need a joint low-dimensional embedding to compare reconstruction against single-modality VAE baselines.

## When NOT to use

- Data is raw IMC/SIMS imagery that has not yet been registered and segmented into per-cell intensities — run notebooks 01–04 first
- Only one modality is available — use the single-modality VAE baseline (protein-only AEp or metabolite-only AEm) instead of the cross-modality joint model
- Cells lack cross-modality correspondence (no single-cell registration via Histone 3/Intercalator IMC and Phosphate 79 m/z TOF-SIMS channels), so protein and metabolite rows cannot be paired per cell

## Inputs

- Segmented single-cell feature matrix with per-cell protein marker intensities (multiplexed IMC channels)
- Per-cell metabolite intensities (TOF-SIMS / 3D-SMF channels)
- Cross-modality registered and segmented cells from notebooks 01–04 (IMC/SIMS registration + single-cell intensity extraction)

## Outputs

- joint_embedding.npy: shared 16-dimensional joint embedding matrix (8-dim protein latent + 8-dim metabolite latent per single cell)
- vae_mae_results.csv: tabulated joint-vs-single-modality reconstruction MAE values per modality branch
- Trained two-branch VAE model (AEp + AEm encoder branches)

## How to apply

Load the segmented single-cell feature matrix (per-cell protein marker intensities and metabolite intensities) produced by the 01–04 pipeline notebooks, then normalize the proteomic and metabolomic feature blocks and route them into the two VAE encoder branches (AEp for protein, AEm for metabolite) of scSpaMet's Keras implementation in the 'scSpaMet' source folder. Each branch encodes to an 8-dimensional latent vector that converges into a shared 16-dimensional joint latent embedding. Train the joint two-branch VAE on the single-cell feature matrix until convergence, recording reconstruction MAE for each modality branch; train protein-only (AEp) and metabolite-only (AEm) single-modality VAEs under matching settings to obtain reference MAE values. The rationale is that a cross-modality VAE handles the imbalance in variability and feature size between the two modalities while preserving per-cell protein-metabolite correspondence established by the registration pipeline.

## Related tools

- **scSpaMet** (Provides the Keras VAE implementation ('scSpaMet' source folder) used to build and train the two-branch joint protein-metabolite embedding) — github.com/coskunlab/ScSpaMet
- **Keras** (Deep learning framework in which the encoder branches AEp (protein) and AEm (metabolite) and the shared 16-dim joint latent embedding are implemented and trained)

## Examples

```
np.save('joint_embedding.npy', joint_vae.encoder.predict([protein_features, metabolite_features]))  # 8-dim latent per branch -> 16-dim joint embedding
```

## Evaluation signals

- The joint latent embedding has exactly 16 dimensions per cell (8 from the protein branch + 8 from the metabolite branch)
- joint_embedding.npy has shape n_cells × 16 and rows correspond one-to-one to segmented single cells from the intensity-extraction notebook outputs
- vae_mae_results.csv reports reconstruction MAE for both the joint two-branch VAE and the matched single-modality baselines (protein-only and metabolite-only) so joint reconstruction can be assessed against them
- Training of the joint VAE runs to convergence on the normalized, modality-split feature blocks before MAE is recorded
- Both modality branches' reconstruction MAE are computed and reported separately, not aggregated into a single loss value

## Limitations

- The two modalities differ in variability and feature size (e.g., more than 200 metabolic markers vs 25 protein markers), which the VAE architecture must explicitly balance
- Metabolite annotation and coverage from TOF-SIMS (lipids, amino acids, metabolic pathways) remains challenging, limiting the metabolite feature block
- TOF-SIMS yields relatively small imaging regions due to the trade-off between imaging speed and spatial resolution, restricting the number of paired single cells

## Evidence

- [abstract] Deep learning-based joint embedding revealed unique metabolite states within cell types: "Deep learning-based joint embedding revealed unique metabolite states within cell types"
- [intro] scSpaMet correlation scope: "scSpaMet enables the correlation of more than 200 metabolic markers and 25 protein markers in individual cells within native tissues."
- [readme] Notebook 05: "05 VAE joint embedding of protein and metabolite modalities"
- [readme] Keras source code: ""scSpaMet" folder contains keras code used for VAE analysis"
