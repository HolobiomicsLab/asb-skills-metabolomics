---
name: exact-structure-accuracy-evaluation
description: Use when a generative model produces candidate molecular structures (SMILES/coordinates)
  from spectra and you need to measure how often the exact target structure is recovered
  and how close near-misses are.
when_to_use_negative:
- Input spectra come from modalities outside IR, Raman, and UV-Vis (e.g. NMR or MS),
  which DiffSpectra does not cover
- You need to evaluate only topological 2D output without any 3D coordinate comparison
  — the RMSD/Hungarian step requires generated 3D coordinates
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DiffSpectra
  role: generates candidate molecules from multi-modal spectra for evaluation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: provides spectral embeddings conditioning the generation being evaluated
  repo: https://huggingface.co/AzureLeon1/DiffSpectra
- name: Hungarian algorithm
  role: atom-to-atom mapping between generated and ground-truth 3D structures before
    RMSD computation
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_content_hash: sha256:b71d5d851376fbaf89347e30ca3034ea44ed91de67db03d2b7e96487d6f2da43
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Hungarian algorithm
  - SpecFormer
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# exact-structure-accuracy-evaluation

## Summary

Evaluates spectra-conditioned molecular generation by exact-match accuracy (top-1/top-10 SMILES match against ground truth) plus fingerprint-based similarity (Tanimoto on Morgan GNN/atom-pair features, functional-group similarity) and 3D quality (RMSD after Hungarian atom mapping). This is how DiffSpectra's 40.76% top-1 / 99.49% top-10 elucidation claims are quantified.

## When to use

Use when a generative model produces candidate molecular structures (SMILES/coordinates) from spectra and you need to measure how often the exact target structure is recovered and how close near-misses are.

## When NOT to use

- Input spectra come from modalities outside IR, Raman, and UV-Vis (e.g. NMR or MS), which DiffSpectra does not cover
- You need to evaluate only topological 2D output without any 3D coordinate comparison — the RMSD/Hungarian step requires generated 3D coordinates

## Inputs

- Generated molecules from DiffSpectra eval (SMILES and 3D coordinates) saved to exp/{exp_name}/eval/
- Ground-truth QM9S test-set molecular structures (2D SMILES and 3D conformations)
- Trained DiffSpectra checkpoint (e.g. exp/allspectra_pretrained_specformer, epoch 40)

## Outputs

- Acc@1 / Acc@10 exact-structure accuracy
- Fingerprint similarity metrics: TaniSim_MG, TaniSim_MA, FGSim, MCES
- Molecular stability, uniqueness, novelty, FCD
- RMSD statistics after Hungarian atom mapping
- results_table.csv in exp/{exp_name}/eval/

## How to apply

Run generation in eval mode (e.g. configs/diffspectra_qm9s.py, --config.eval.num_samples 10000, --config.eval.ckpts '40') and save generated molecules; accuracy and similarity metrics are computed automatically alongside generation, or manually via eval_sampled_mols/base_metrics.py, which matches generated SMILES against ground truth to yield Acc@1 and Acc@10 plus molecular stability, uniqueness, novelty, and FCD. For fingerprint similarity, compare generated SMILES against the ground-truth structure using Tanimoto similarity on Morgan-graph (TaniSim_MG), atom-map (TaniSim_MA) fingerprints and functional-group similarity (FGSim), as reported in Table 2 (e.g. TaniSim_MA 0.8350→0.9262 when adding 3D). For 3D quality, run eval_sampled_mols/rmsd.py, which performs atom-to-atom mapping against ground-truth 3D structures using the Hungarian algorithm and reports root mean square deviation. Aggregate all metrics into a single results_table.csv in exp/{exp_name}/eval/ with rows per variant and columns for top-1/top-10 accuracy, similarity, and RMSD.

## Related tools

- **DiffSpectra** (generates candidate molecules from multi-modal spectra for evaluation) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (provides spectral embeddings conditioning the generation being evaluated) — https://huggingface.co/AzureLeon1/DiffSpectra
- **Hungarian algorithm** (atom-to-atom mapping between generated and ground-truth 3D structures before RMSD computation)

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Acc@1 ≈ 40.76% and Acc@10 ≈ 99.49% on the QM9S test set (within diffusion sampling tolerance)
- Molecular stability ≈ 98.2% and FCD ≈ 0.088, consistent with reported values
- exp/{exp_name}/eval/ contains generated molecule files and metric outputs
- 2D-only ablation shows the reported drop (top-1 9.49% vs 40.76% joint 2D&3D) with worsened MCES and similarity metrics
- RMSD is computed only after Hungarian atom mapping against ground-truth 3D structures

## Limitations

- Validation is primarily computational; no experimental/laboratory validation has been performed
- Results depend on stochastic diffusion sampling, so exact accuracies are reproducible only within run-to-run tolerance
- Framework limited to three spectral types (UV-Vis, IR, Raman); NMR and MS not covered

## Evidence
