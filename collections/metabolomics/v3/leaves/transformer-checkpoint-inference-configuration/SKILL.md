---
name: transformer-checkpoint-inference-configuration
description: Use when a trained or pretrained NMR2Struct checkpoint and its alphabet.npy
  must be wired up for inference — SMILES generation from spectra or from 957-substructure
  label vectors — and correct tgt_start/tgt_stop token IDs must be set in the YAML.
when_to_use_negative:
- You want to train a model from scratch — use nmr_train with the training YAML, not
  nmr_infer
- The molecule exceeds 19 heavy atoms or contains elements outside C, N, O (plus H)
  — the framework was demonstrated only on molecules with up to 19 heavy atoms from
  SpectraBase containing only C, N, and O as non-hydrogen atoms
tools:
- name: NMR2Struct
  role: Package providing the nmr_infer CLI, YAML config schema, token conventions,
    and checkpoint loading for inference
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer
  role: Encoder-decoder architecture whose pretrained weights initialize the multitask
    model and whose decoder generates 15 SMILES predictions per input via top-k sampling
    (k=5)
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: Anaconda
  role: Package manager used to create the NMR_env environment from environment.yml
    before running inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:5dd1994e19b88fcf0fca09901bb27de65faf1741cfedce1ae637434fb302e086
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - convolutional neural network (CNN)
  - transformer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transformer-checkpoint-inference-configuration

## Summary

Configures alphabet/token conventions and runs inference with a pretrained NMR2Struct transformer or multitask spectrum-to-structure checkpoint via YAML configs and the nmr_infer CLI, generating 15 SMILES predictions per input spectrum by top-k sampling (k=5).

## When to use

Use when a trained or pretrained NMR2Struct checkpoint and its alphabet.npy must be wired up for inference — SMILES generation from spectra or from 957-substructure label vectors — and correct tgt_start/tgt_stop token IDs must be set in the YAML.

## When NOT to use

- You want to train a model from scratch — use nmr_train with the training YAML, not nmr_infer
- The molecule exceeds 19 heavy atoms or contains elements outside C, N, O (plus H) — the framework was demonstrated only on molecules with up to 19 heavy atoms from SpectraBase containing only C, N, and O as non-hydrogen atoms

## Inputs

- Trained model checkpoint (multitask spectrum-to-structure or substructure-to-structure transformer)
- alphabet.npy token-alphabet file from training (or example_configs/alphabet.npy)
- YAML config with global_args, data, model, and inference sections
- Test-split simulated spectra as hdf5 ('spectra' dataset: concatenated 1H interpolated to 28000 points over -2 to 12 ppm plus 13C binned into 80 bins spanning 3.42 to 231.3 ppm)
- Splits file specifying which dataset indices belong to the test set
- For single-spectrum inference: (ppm, intensity) text files for 1H and a comma-separated carbon ppm list for 13C

## Outputs

- Raw predictions hdf5 file written to global_args.save_dir (15 SMILES predictions per test-set spectrum)
- Substructure predictions hdf5 when pred_gen_fn is infer_multitask_model_substructures

## How to apply

Clone MarklandGroup/NMR2Struct, install via the provided environment.yml (conda activate NMR_env, pip install -e .), and adapt an example_configs YAML (training_substruct_to_struct.yaml or training_spectrum_to_struct.yaml) into an inference config. Set inference.run_inference_args.pred_gen_opts.alphabet to the alphabet.npy produced during training (or example_configs/alphabet.npy to reproduce the paper's model), and set tgt_start_token and tgt_stop_token to length of alphabet + 1 and + 2 per the token conventions; for the multitask model structure_model_args.source_size must be 28045 (the concatenated interpolated 1H/13C spectrum length). Run `nmr_infer config.yaml 0 1`, where 0 and 1 are local rank and GPU process count, ensuring inference.splits points to the file marking test-set indices; raw predictions are written as hdf5 to global_args.save_dir. To infer substructures instead of SMILES, switch pred_gen_fn to 'infer_multitask_model_substructures'. For a single example, use nmr_infer_single_spectrum with --hnmr_file, --cnmr_file, --ckpt and --normalize.

## Related tools

- **NMR2Struct** (Package providing the nmr_infer CLI, YAML config schema, token conventions, and checkpoint loading for inference) — https://github.com/MarklandGroup/NMR2Struct
- **transformer** (Encoder-decoder architecture whose pretrained weights initialize the multitask model and whose decoder generates 15 SMILES predictions per input via top-k sampling (k=5)) — https://github.com/MarklandGroup/NMR2Struct
- **Anaconda** (Package manager used to create the NMR_env environment from environment.yml before running inference)

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- tgt_start_token and tgt_stop_token equal length of alphabet + 1 and length of alphabet + 2 respectively, and tgt_pad_token equals length of alphabet
- For the multitask model, structure_model_args.source_size is exactly 28045
- Exactly 15 SMILES predictions are generated per test-set spectrum with top-k sampling, k = 5
- Raw predictions hdf5 file appears in the global_args save directory alongside checkpoints and tensorboard files
- Canonicalized predictions can be split into valid_predictions and bad_predictions with RDKit, and the exact-match accuracy is reproducible against the paper benchmark (69.6% top-15 spectrum-to-structure; 93.2% substructure-to-structure)

## Limitations

- The inference workflow with full datasets is intended only for datasets already split during training (e.g. reproducing test-set metrics); single examples need the nmr_infer_single_spectrum entry point
- Token/alphabet fields are auto-populated during training, so inferring without training requires manually applying the README token conventions; mismatched alphabet files will silently mis-map SMILES tokens
- The paper's checkpoint was trained on simulated SpectraBase/GDB-17 spectra that do not fully capture nuances of real spectra
- No software or package version numbers are stated for NMR2Struct, conda, or TensorBoard in the README excerpt

## Evidence

- [other] tgt_start_token and tgt_stop_token to length of alphabet + 1 and length of alphabet + 2 respectively per the token conventions: "inference.run_inference_args.pred_gen_opts.tgt_start_token | length of alphabet + 1"
- [readme] nmr_infer CLI invocation with rank and GPU process count: "Once the alphabet is set, just run:
```
nmr_infer config.yaml 0 1
```"
- [readme] paper's original alphabet must be supplied from example_configs: "If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy``` instead."
- [results] generation protocol: top-k sampling with k=5 and 15 predictions per input: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] pretraining rationale: multitask transformer is initialized from substructure-to-structure weights: "an important component of training this multitask spectrum-to-structure model is the initialization of the transformer using the weights obtained from training on the substructure-to-structure task"
