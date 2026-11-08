---
name: reconstruction-error-evaluation-mae
description: Use when training a joint multi-branch VAE (e.g., protein + metabolite
  single-cell features) and you need a quantitative reconstruction metric to validate
  the shared latent embedding against single-modality baselines.
when_to_use_negative:
- Input is already a reduced embedding or clustering result rather than raw per-cell
  protein/metabolite intensity features — reconstruction error is not meaningful without
  reconstructable feature blocks
- Modalities come from different cells (unpaired data) — the joint two-branch VAE
  assumes protein and metabolite profiles from the same single cell
- You need cell-type label accuracy rather than reconstruction fidelity — use segmentation/clustering
  outputs (notebooks 03-04) instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: scSpaMet
  role: Keras implementation of the two-branch VAE (AEp protein encoder, AEm metabolite
    encoder) whose reconstruction MAE is computed and benchmarked
  repo: github.com/coskunlab/ScSpaMet
- name: Keras
  role: Framework used to build and train the joint and single-modality VAE models
    for MAE evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:10e928861719c7ad7bf24fff21bd6489718f980912c846fd47694a747fc8f9db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - scSpaMet
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# reconstruction-error-evaluation-mae

## Summary

Evaluates cross-modality VAE joint embedding quality by computing per-modality reconstruction MAE for the joint two-branch protein-metabolite VAE and comparing it against single-modality baseline VAEs trained under matching settings.

## When to use

Use when training a joint multi-branch VAE (e.g., protein + metabolite single-cell features) and you need a quantitative reconstruction metric to validate the shared latent embedding against single-modality baselines.

## When NOT to use

- Input is already a reduced embedding or clustering result rather than raw per-cell protein/metabolite intensity features — reconstruction error is not meaningful without reconstructable feature blocks
- Modalities come from different cells (unpaired data) — the joint two-branch VAE assumes protein and metabolite profiles from the same single cell
- You need cell-type label accuracy rather than reconstruction fidelity — use segmentation/clustering outputs (notebooks 03-04) instead

## Inputs

- Per-cell segmented feature matrix with protein marker intensities and metabolite intensities from the intensity-extraction notebook (01-04 pipeline outputs)
- Normalized proteomic and metabolomic feature blocks split into the two VAE modality branches
- Trained joint two-branch VAE (scSpaMet scSpaMet Keras implementation, AEp/AEm encoders)
- Trained single-modality baseline VAEs (protein-only AEp, metabolite-only AEm)

## Outputs

- vae_mae_results.csv — joint-vs-single-modality reconstruction MAE table per modality
- Per-modality reconstruction MAE values recorded during joint VAE training convergence

## How to apply

After training the scSpaMet Keras two-branch VAE — encoder branch AEp for normalized protein marker intensities and branch AEm for normalized metabolite intensities converging into a shared 16-dim joint latent (8 per modality) — record the reconstruction MAE for each modality branch at convergence. Then train the single-modality baseline VAEs (protein-only AEp and metabolite-only AEm) under matching settings to obtain reference MAE values. Tabulate joint-vs-single-modality MAE values in vae_mae_results.csv so the joint embedding can be assessed against single-modality reconstruction; accept the joint model only when per-modality MAE is computed and reported for both the joint and baseline models.

## Related tools

- **scSpaMet** (Keras implementation of the two-branch VAE (AEp protein encoder, AEm metabolite encoder) whose reconstruction MAE is computed and benchmarked) — github.com/coskunlab/ScSpaMet
- **Keras** (Framework used to build and train the joint and single-modality VAE models for MAE evaluation)

## Examples

```
# inside notebook 05 (VAE joint embedding of protein and metabolite modalities)
mae_table.to_csv('vae_mae_results.csv')
```

## Evaluation signals

- vae_mae_results.csv exists and contains reconstruction MAE rows for both the joint two-branch VAE and both single-modality baselines (protein-only AEp, metabolite-only AEm)
- MAE is reported separately per modality branch (protein and metabolite), not only as a single aggregate loss
- Baseline VAEs were trained under settings matching the joint model, making the MAE comparison valid
- Training ran to convergence for all models before MAE was recorded
- The joint latent dimensionality matches the expected 16 dims (8 per modality) in the saved joint_embedding.npy

## Limitations

- The paper notes the imbalance in variability and feature size between proteomic and metabolomic modalities, which must be handled by normalization before meaningful per-branch MAE comparison
- Reconstruction MAE measures fidelity of the shared 16-dim embedding but does not directly certify that the embedding preserves biologically meaningful metabolite states; downstream analyses (metabolite competition, trajectory inference) are separate validation steps
- The skill is specific to paired single-cell protein-metabolite profiles from the scSpaMet 3D-SMF + IMC pipeline; MAE comparison on other modality pairs is untested here

## Evidence

- [abstract] Deep learning-based joint embedding revealed unique metabolite states within cell types: "Deep learning-based joint embedding revealed unique metabolite states within cell types"
- [results] workflow finding: [evidence span withheld — anchor status: not_found]
- [readme] README notebook listing: "05 VAE joint embedding of protein and metabolite modalities"
- [readme] README source code listing: ""scSpaMet" folder contains keras code used for VAE analysis"
