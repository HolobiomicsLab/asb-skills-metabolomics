---
name: ablation-experiment-comparison-analysis
description: 'Use when reproducing an ablation that isolates a component''s contribution:
  you have two checkpoints (pre-trained encoder vs. random init) for the same conditional
  generation task and need paired deltas on shared metrics such as top-1/top-10 accuracy
  and FCD.'
when_to_use_negative:
- The goal is to compare different spectra modalities (IR-only vs. Raman-only vs.
  UV-Vis-only) rather than to isolate the effect of encoder initialization — that
  is a separate multi-modal conditioning ablation.
- You lack a pre-trained encoder checkpoint for the component in question; with only
  a randomly initialized model you cannot form the pre-trained/untrained contrast.
- You need experimental/laboratory validation rather than computational reproduction
  — the DiffSpectra validation is stated to be primarily computational.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3332
tools:
- name: DiffSpectra
  role: Conditional diffusion framework whose eval mode generates molecules conditioned
    on multi-modal spectra; run twice, once per SpecFormer initialization variant
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder whose pre-trained vs. randomly initialized
    weights form the two arms of the ablation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network held fixed across both ablation arms so
    the SpecFormer initialization is the only differing factor
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:4ce1985f0db48e84d5a594d597497395a31025ac33309d6df488b942465d566d
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - SpecFormer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ablation-experiment-comparison-analysis

## Summary

Quantifies the contribution of a pre-trained encoder by re-running a conditional generative pipeline with two initialization variants (pre-trained vs. randomly initialized) and comparing paired metrics side-by-side, as done for SpecFormer in DiffSpectra's spectral structure elucidation ablation.

## When to use

Use when reproducing an ablation that isolates a component's contribution: you have two checkpoints (pre-trained encoder vs. random init) for the same conditional generation task and need paired deltas on shared metrics such as top-1/top-10 accuracy and FCD.

## When NOT to use

- The goal is to compare different spectra modalities (IR-only vs. Raman-only vs. UV-Vis-only) rather than to isolate the effect of encoder initialization — that is a separate multi-modal conditioning ablation.
- You lack a pre-trained encoder checkpoint for the component in question; with only a randomly initialized model you cannot form the pre-trained/untrained contrast.
- You need experimental/laboratory validation rather than computational reproduction — the DiffSpectra validation is stated to be primarily computational.

## Inputs

- Hugging Face model checkpoints: DiffSpectra with pre-trained SpecFormer weights (specformer.ckpt) and DiffSpectra with randomly initialized SpecFormer
- QM9S dataset (QM9 augmented with simulated IR, Raman, and UV-Vis spectra) placed at the path configured in configs/ as data.root
- Evaluation config configs/diffspectra_qm9s.py with configs/base_qm9.py

## Outputs

- specformer_ablation_metrics.csv — side-by-side table of top-1 accuracy, top-10 accuracy, and FCD for pre-trained vs. random SpecFormer, with deltas
- Generated molecules and metrics per variant at exp/{exp_name}/eval/

## How to apply

Load both configurations from the Hugging Face checkpoints (DiffSpectra with pre-trained SpecFormer weights in specformer.ckpt, and DiffSpectra with a randomly initialized SpecFormer) and generate molecules with the evaluation script in 'eval' mode (--mode eval, eval.num_samples 10000, eval.save_mols true) conditioned on the multi-modal spectra (IR, Raman, UV-Vis) of the QM9S dataset. Run the identical generation procedure for both variants so the only differing factor is the SpecFormer initialization; pre-training was done with masked patches reconstruction plus InfoNCE contrastive alignment on the QM9S training set only, to avoid data leakage. Then compute top-1 accuracy, top-10 accuracy, and Fréchet ChemNet Distance (FCD) for each variant with eval_sampled_mols/base_metrics.py, matching generated molecules to ground-truth structures by SMILES identity. Tabulate the two variants side-by-side in a CSV (specformer_ablation_metrics.csv) and compute the deltas, targeting the paper's reported +9.49 percentage points in top-1 accuracy (40.76% vs. 35.64%) and an FCD delta of about 1.651.

## Related tools

- **DiffSpectra** (Conditional diffusion framework whose eval mode generates molecules conditioned on multi-modal spectra; run twice, once per SpecFormer initialization variant) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder whose pre-trained vs. randomly initialized weights form the two arms of the ablation) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network held fixed across both ablation arms so the SpecFormer initialization is the only differing factor) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Pre-trained-encoder arm reproduces the paper's top-1 accuracy of ~40.76% and the untrained arm ~35.64%, giving a delta of ≈+9.49 percentage points
- FCD delta between variants matches the paper's reported ≈1.651 within reproduction tolerance
- Both arms use the same eval settings: eval.num_samples 10000, eval.ckpts '40', spectra_version allspectra
- Top-10 accuracy stays near the reported ~99.49% level for the full-configuration arm, confirming the SMILES-identity matching is functioning
- Metrics and generated molecules exist at exp/{exp_name}/eval/ for both workdirs before the CSV is written

## Limitations

- Evaluation is stochastic diffusion sampling on 10,000 samples; modest run-to-run variance in top-1 accuracy and FCD is expected relative to reported values.
- The ablation is scoped to three spectral types (UV-Vis, IR, Raman); results do not transfer to NMR or mass spectrometry.
- No software versions, model hyperparameters, or deep-learning-library versions are stated, so exact reproduction may require tuning the environment from env.sh.
- Conclusion applies to the QM9S benchmark; pre-training benefit on other spectral datasets is unverified.

## Evidence

- [results] Equipping DiffSpectra with a pre-trained SpecFormer raises top-1 structure elucidation accuracy to 40.76%, compared to 35.64% when the SpecFormer is not pre-trained.: [evidence span withheld — anchor status: not_found]
- [abstract] Extensive experiments demonstrate that DiffSpectra accurately elucidates molecular structures, achieving 40.76% top-1 and 99.49% top-10 accuracy.: "Extensive experiments demonstrate that DiffSpectra accurately elucidates molecular structures, achieving 40.76% top-1 and 99.49% top-10 accuracy."
- [methods] In the second stage, SpecFormer is pre-trained exclusively on the QM9S training set to avoid data leakage.: "In the second stage, SpecFormer is pre-trained exclusively on the QM9S training set to avoid data leakage."
- [readme] compute metrics based on the generated molecules. For accuracy and similarity evaluation, these metrics will be computed automatically: "After generating molecules, compute metrics based on the generated molecules. For accuracy and similarity evaluation, these metrics will be computed automatically along with the generation process."
