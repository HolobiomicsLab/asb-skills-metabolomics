---
name: top-k-sampled-smiles-generation
description: Use when you have a trained NMR2Struct checkpoint (substructure-to-structure
  transformer or multitask model), a loaded alphabet.npy, and need per-input candidate
  SMILES sets sized for top-15 exact-match evaluation.
when_to_use_negative:
- You need deterministic single-structure output rather than a ranked candidate set
  — this protocol deliberately samples 15 candidates with k=5 nondeterminism
- Inferring substructures rather than SMILES — use pred_gen_fn 'infer_multitask_model_substructures'
  instead of the SMILES-generation inference configuration
- Molecules beyond the model's training scope (heavier than 19 heavy atoms, elements
  beyond C, N, O as non-hydrogen atoms, or stereochemistry-dependent queries)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: NMR2Struct
  role: Provides the nmr_infer entry point and YAML config system that implement top-k
    sampling-based SMILES generation and write raw predictions to hdf5
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: Decoder whose token logits are sampled with top-k (k=5) to autoregressively
    emit SMILES tokens bounded by tgt_start_token and tgt_stop_token
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes the generated SMILES strings so exact-match scoring against
    canonical target SMILES is well-defined
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:abbdf983336a18e171e3beaca35c1b82b7ee33784039505f4d698b3cbbfb19c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - convolutional neural network (CNN)
  - transformer (encoder-decoder)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# top-k-sampled-smiles-generation

## Summary

Generates candidate molecular structures as SMILES strings by sampling from a transformer decoder (structure-assembly or multitask spectrum-to-structure model) using top-k random sampling with k=5 and emitting 15 predictions per input, which is the decoding protocol underlying the paper's reported exact-match accuracies.

## When to use

Use when you have a trained NMR2Struct checkpoint (substructure-to-structure transformer or multitask model), a loaded alphabet.npy, and need per-input candidate SMILES sets sized for top-15 exact-match evaluation.

## When NOT to use

- You need deterministic single-structure output rather than a ranked candidate set — this protocol deliberately samples 15 candidates with k=5 nondeterminism
- Inferring substructures rather than SMILES — use pred_gen_fn 'infer_multitask_model_substructures' instead of the SMILES-generation inference configuration
- Molecules beyond the model's training scope (heavier than 19 heavy atoms, elements beyond C, N, O as non-hydrogen atoms, or stereochemistry-dependent queries)

## Inputs

- Trained NMR2Struct model checkpoint (substructure-to-structure transformer or multitask spectrum-to-structure model)
- alphabet.npy token alphabet from training (or example_configs/alphabet.npy)
- YAML configuration file with global_args, model, data, and inference sections
- Test-set spectra hdf5 (concatenated 1H interpolated to 28000 points, -2 to 12 ppm at 0.0005 ppm resolution; 13C in 80 bins over 3.42 to 231.3 ppm) or binary substructure label hdf5
- Splits file or train/val/test fractions specifying the evaluation subset

## Outputs

- Raw predictions hdf5 file containing 15 generated SMILES strings per input
- Canonicalized predictions hdf5 (valid_predictions and bad_predictions groups) after RDKit processing
- Per-input ranked candidate lists enabling top-15 exact-structure accuracy computation

## How to apply

Configure inference.pred_gen_opts in the YAML config, setting tgt_start_token to length of alphabet + 1 and tgt_stop_token to length of alphabet + 2 per the README token convention, and point the alphabet field at the alphabet.npy saved during training (or example_configs/alphabet.npy for the original paper model). Top-k random sampling with k = 5 generates SMILES from the decoder, with 15 predictions produced per input spectrum or substructure vector; the inference entry point nmr_infer writes raw predictions to an hdf5 file in the save directory under global_args. Afterwards, canonicalize predictions with RDKit (nmr_analyze, analysis_type 'SMILES') so exact-match scoring against canonical target SMILES is meaningful, since nondeterministic sampling means the correct molecule may appear at any rank among the 15 predictions. The multitask model initialized from pretrained transformer weights (structure_model_ckpt) is required for spectrum inputs; the decoder-only configuration applies to the substructure-to-structure task.

## Related tools

- **NMR2Struct** (Provides the nmr_infer entry point and YAML config system that implement top-k sampling-based SMILES generation and write raw predictions to hdf5) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (Decoder whose token logits are sampled with top-k (k=5) to autoregressively emit SMILES tokens bounded by tgt_start_token and tgt_stop_token) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes the generated SMILES strings so exact-match scoring against canonical target SMILES is well-defined)

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- Each input yields exactly 15 predictions, and tgt_start_token / tgt_stop_token equal length of alphabet + 1 and + 2 respectively
- Top-15 exact-match accuracy on the test set reproduces the reported benchmark of 69.6% for the multitask model
- Substructure-to-structure decoding reproduces the reported 93.2% test accuracy (96.0% at 10 heavy atoms, dropping below 80% at 18-19 heavy atoms)
- Bad SMILES rate is nonzero and expected: invalid decodes are routed to bad_predictions by RDKit canonicalization rather than crashing inference
- Raw predictions hdf5 appears in the global_args save directory with write_freq-consistent checkpointing

## Limitations

- Sampling-based decoding is stochastic; accuracy is defined over the first 15 predictions, not the single top-1 decode
- Accuracy depends on transformer pretraining on the substructure-to-structure task; without pretrained weights initialization quality may degrade
- Evaluation is on simulated spectra, which do not fully capture the nuances of real spectra; model does not predict stereochemistry

## Evidence

- [results] top-k random sampling was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [readme] the target tokens are bounded by start/stop tokens placed just beyond the alphabet: [evidence span withheld — anchor status: not_found]
- [abstract] headline accuracy figure motivating the 15-candidate decoding budget: "we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions, reducing the search space by up to 11 orders of magnitude."
