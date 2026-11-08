---
name: augmentation-ablation-accuracy-comparison
description: Use when you need to quantify how much of a diffusion-based molecular
  generator's accuracy comes from built-in equivariance versus training-time data
  augmentation, i.e., when ablating an augmentation flag from a config and re-evaluating.
when_to_use_negative:
- The goal is to evaluate DiffSpectra's headline elucidation performance with the
  full pretrained SpecFormer configuration rather than isolating the augmentation
  contribution — use the standard allspectra_pretrained_specformer evaluation instead.
- The spectral modality is a single type (ir, raman, or uv only) — the equivariance-vs-augmentation
  comparison was established under the all-spectra setting and its reference values
  (20.25%, 31.24%, 35.64%) do not apply to single-modality runs.
- The task involves NMR or mass spectrometry data — the article states DiffSpectra
  currently focuses only on UV-Vis, IR, and Raman spectra.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose augmentation flag is ablated; run
    in eval mode to generate structures conditioned on spectra
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework providing the evaluation pipeline
    (main.py, configs/diffspectra_qm9s.py, eval_sampled_mols/base_metrics.py)
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder producing embeddings that condition the diffusion process
    during the ablation runs
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:3f0e671592b0051c76c91e4f8caa04c7fe242535786a4a9921e91c129b297bf5
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# augmentation-ablation-accuracy-comparison

## Summary

An ablation protocol for DiffSpectra/DMT that isolates the contribution of SE(3)-equivariant architecture versus rotation/translation data augmentation by re-running spectra-conditioned generation with augmentation removed and comparing top-1 accuracy and MCES against the with-augmentation baseline.

## When to use

Use when you need to quantify how much of a diffusion-based molecular generator's accuracy comes from built-in equivariance versus training-time data augmentation, i.e., when ablating an augmentation flag from a config and re-evaluating.

## When NOT to use

- The goal is to evaluate DiffSpectra's headline elucidation performance with the full pretrained SpecFormer configuration rather than isolating the augmentation contribution — use the standard allspectra_pretrained_specformer evaluation instead.
- The spectral modality is a single type (ir, raman, or uv only) — the equivariance-vs-augmentation comparison was established under the all-spectra setting and its reference values (20.25%, 31.24%, 35.64%) do not apply to single-modality runs.
- The task involves NMR or mass spectrometry data — the article states DiffSpectra currently focuses only on UV-Vis, IR, and Raman spectra.

## Inputs

- QM9S dataset from HuggingFace (AzureLeon1/DiffSpectra), spectra_version=all spectra
- Trained DMT model checkpoint 40 from huggingface.co/AzureLeon1/DiffSpectra
- DiffSpectra configuration file configs/diffspectra_qm9s.py with data.root set to the local dataset path
- Augmentation setting flag in the DMT configuration (toggled off for the ablation run)

## Outputs

- Top-1 accuracy (Acc@1) for the with-augmentation condition over 10,000 generated samples
- Top-1 accuracy for the augmentation-removed (architecture-only) condition over 10,000 generated samples
- MCES values for both conditions (reference: 1.7795 with augmentation, 3.6036 without)
- results_table.csv containing both accuracy values plus the reference 20.25% figure

## How to apply

Load the QM9S dataset (HuggingFace deposit AzureLeon1/DiffSpectra) and DMT checkpoint 40, configure spectra-conditioned generation with spectra_version=all spectra as in configs/diffspectra_qm9s.py. Run the reverse denoising process (continuous-time diffusion from t=1 to t=0) with DMT in eval mode for num_samples=10000 to obtain the baseline top-1 accuracy with augmentation (reference: 31.24% for the data-based variant, 35.64% for the model-based SE(3)-equivariant variant with MCES 1.7795). Then remove the augmentation setting from the DMT configuration and rerun the identical evaluation (eval mode, num_samples=10000, checkpoint 40) to obtain the architecture-only condition. Validate by checking that the augmentation-removed accuracy matches the reported 20.25% top-1 (MCES 3.6036) and that the with-augmentation condition exceeds it, confirming architectural equivariance plus augmentation outperforms augmentation removal. Write both conditions plus the reference value to results_table.csv.

## Related tools

- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose augmentation flag is ablated; run in eval mode to generate structures conditioned on spectra) — https://github.com/AzureLeon1/DiffSpectra
- **DiffSpectra** (Spectra-conditioned diffusion framework providing the evaluation pipeline (main.py, configs/diffspectra_qm9s.py, eval_sampled_mols/base_metrics.py)) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder producing embeddings that condition the diffusion process during the ablation runs)

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.data.spectra_version allspectra
```

## Evaluation signals

- Augmentation-removed condition reproduces the reported 20.25% top-1 accuracy (checkpoint 40, num_samples=10000)
- With-augmentation condition exceeds the augmentation-removed accuracy, confirming architectural equivariance plus augmentation outperforms augmentation removal
- MCES for the augmentation-removed condition approaches the reported 3.6036 (vs 1.7795 with augmentation)
- results_table.csv contains exactly three values: both conditions plus the 20.25% reference figure
- Both runs use identical settings otherwise (eval mode, checkpoint 40, spectra_version=all spectra) so the only difference is the augmentation flag

## Limitations

- Removing augmentation substantially degrades performance (20.25% Acc@1, MCES 3.6036), so results are sensitive to this configuration toggle and should not be reported interchangeably with the full model.
- The comparison is only meaningful when the identical checkpoint (40) and sample count (10000) are used for both conditions.
- Validation is primarily computational; the article notes 'our validation to date has been primarily computational' with no experimental/laboratory validation.

## Evidence

- [results] model-based vs data augmentation ablation rationale: "The denoising process is parameterized by the Diffusion Molecule Transformer, which preserves permutation and SE(3) equivariance"
- [methods] reverse denoising procedure used in both ablation runs: "The reverse-time denoising process starts from a standard Gaussian sample"
- [readme] README evaluation command specifying checkpoint and sample count for the ablation reruns: "--config.eval.ckpts '40' \
    --config.eval.num_samples 10000 \
    --config.eval.save_mols true"
- [readme] headline accuracy context for the evaluation metric used: "Extensive experiments demonstrate that DiffSpectra achieves 40.76% top-1 accuracy and 99.49% top-10 accuracy in recovering exact molecular structures."
