---
name: nmr-spectrum-inference-decoding
description: Use when you have a trained multitask spectrum-to-structure checkpoint
  and simulated 1H/13C NMR spectra (concatenated 28000-point 1H grid plus 80-bin 13C
  shifts) and need SMILES structure predictions without molecular formula input.
when_to_use_negative:
- You only need substructure predictions, not full SMILES structures — use pred_gen_fn
  'infer_multitask_model_substructures' instead of SMILES generation
- You are running the substructure-to-structure transformer alone (no spectral input);
  that model uses a different source representation (957 substructure indices) and
  its own token conventions
- Input is experimental (not simulated) NMR spectra — the model was trained on MestreNova-simulated
  SpectraBase spectra and does not yet capture nuances of real spectra
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Provides nmr_infer entry point that decodes spectra into 15 SMILES predictions
    via top-k sampling (k=5), and nmr_analyze for canonicalizing raw predictions with
    RDKit
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: Pretrained structure-assembling decoder that autoregressively generates SMILES
    tokens from spectral/substructure features during inference
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes the 15 generated SMILES strings per spectrum so exact-match
    accuracy against canonical target SMILES can be computed
- name: Anaconda
  role: Package manager used to create the NMR_env environment via environment.yml
    before running inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:5a6189153d1d5a88474442a61c7773e76e530374b3c2c493e763eada0eebc4fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - convolutional neural network (CNN)
  - transformer (encoder-decoder)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nmr-spectrum-inference-decoding

## Summary

Run inference with the NMR2Struct spectrum-to-structure multitask model to decode 1D 1H/13C NMR spectra into SMILES structure predictions, using the correct alphabet/token conventions and top-k sampling, then canonicalize outputs for downstream accuracy analysis.

## When to use

Use when you have a trained multitask spectrum-to-structure checkpoint and simulated 1H/13C NMR spectra (concatenated 28000-point 1H grid plus 80-bin 13C shifts) and need SMILES structure predictions without molecular formula input.

## When NOT to use

- You only need substructure predictions, not full SMILES structures — use pred_gen_fn 'infer_multitask_model_substructures' instead of SMILES generation
- You are running the substructure-to-structure transformer alone (no spectral input); that model uses a different source representation (957 substructure indices) and its own token conventions
- Input is experimental (not simulated) NMR spectra — the model was trained on MestreNova-simulated SpectraBase spectra and does not yet capture nuances of real spectra

## Inputs

- Trained multitask spectrum-to-structure model checkpoint (structure_model_ckpt)
- alphabet.npy token alphabet from training (or example_configs/alphabet.npy for the paper model)
- Test-split spectra hdf5 file(s) with dataset 'spectra': concatenated interpolated 1H NMR (28000 points, -2 to 12 ppm) and 80-bin 13C shifts (3.42 to 231.3 ppm)
- YAML configuration file with global_args, model, data, and inference sections
- Splits file specifying test-set indices
- Target SMILES and substructure_957 label files for evaluation (from Zenodo record 13892026)

## Outputs

- Raw predictions hdf5 file (predictions_dataset_0_*.h5) containing 15 generated SMILES per test spectrum
- processed_predictions.h5 with canonicalized predictions split into valid_predictions and bad_predictions groups
- Top-15 exact-structure accuracy on the test set (benchmark: 69.6%)

## How to apply

Load the multitask model checkpoint together with the alphabet.npy produced during training (use example_configs/alphabet.npy for the original paper model), and set inference.run_inference_args.pred_gen_opts.tgt_start_token and tgt_stop_token to length of alphabet + 1 and + 2 per the README token conventions. Point the config's data section at the hdf5 spectra files (dataset 'spectra', each row a concatenated 1H/13C vector, 1H interpolated to 28000 points over -2 to 12 ppm at 0.0005 ppm resolution; 13C binned into 80 bins from 3.42 to 231.3 ppm). Run `nmr_infer config.yaml 0 1` with the inference section configured for SMILES generation (default pred_gen_fn generates SMILES; set inference.splits to the test-set indices file and sets_to_run to ['test']), which generates 15 SMILES per spectrum via top-k sampling with k = 5. Process the raw predictions hdf5 with nmr_analyze using analysis_type 'SMILES', pattern 'predictions_dataset_0_[0-9]+.h5', and f_addn_args pointing to substructures_957.p, which canonicalizes SMILES with RDKit and splits outputs into valid_predictions and bad_predictions in processed_predictions.h5. Judge success by whether the canonicalized target SMILES appears among the 15 predictions — the reported test-set top-15 exact-match accuracy is 69.6%.

## Related tools

- **NMR2Struct** (Provides nmr_infer entry point that decodes spectra into 15 SMILES predictions via top-k sampling (k=5), and nmr_analyze for canonicalizing raw predictions with RDKit) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (Pretrained structure-assembling decoder that autoregressively generates SMILES tokens from spectral/substructure features during inference) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes the 15 generated SMILES strings per spectrum so exact-match accuracy against canonical target SMILES can be computed)
- **Anaconda** (Package manager used to create the NMR_env environment via environment.yml before running inference)

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- tgt_start_token equals len(alphabet)+1 and tgt_stop_token equals len(alphabet)+2 (README token convention table)
- Exactly 15 SMILES predictions are generated per test spectrum via top-k sampling with k=5
- nmr_analyze produces processed_predictions.h5 with valid_predictions and bad_predictions groups and canonicalized SMILES
- Computed top-15 exact-match accuracy on the test set reproduces the reported benchmark of 69.6%
- All canonicalized predictions parse as valid molecules; invalid ones land in bad_predictions

## Limitations

- The multitask model was trained and benchmarked on simulated MestreNova spectra of molecules with up to 19 heavy atoms containing only C, N, O; accuracy on real experimental spectra is not established
- The model does not predict stereochemistry
- Inference with nmr_infer on the full dataset expects the dataset used for training; for single spectra use the nmr_infer_single_spectrum entry point with --normalize

## Evidence

- [results] top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm with a resolution of 0.0005 ppm.: "linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm"
- [readme] inference.run_inference_args.pred_gen_opts.tgt_start_token | length of alphabet + 1: "inference.run_inference_args.pred_gen_opts.tgt_start_token | length of alphabet + 1"
- [readme] If you want to use the alphabet that was used in the original model from the paper, put the path to the file example_configs/alphabet.npy instead.: "If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy``` instead."
- [abstract] we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions: "we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions"
- [methods] Data available on Zenodo at: https://zenodo.org/records/13892026: "Data available on Zenodo at: https://zenodo.org/records/13892026"
