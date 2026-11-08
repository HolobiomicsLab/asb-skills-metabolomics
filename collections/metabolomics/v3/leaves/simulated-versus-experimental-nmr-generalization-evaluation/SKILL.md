---
name: simulated-versus-experimental-nmr-generalization-evaluation
description: Use when a model trained on simulated 1H and/or 13C NMR spectra must
  be evaluated on real experimental spectra, and the simulated-to-experimental accuracy
  gap needs to be quantified (e.g., top-15 exact-match rate ~33.0% vs 69.6% simulated).
when_to_use_negative:
- Input spectra are simulated MestreNova-predicted SpectraBase spectra — use the standard
  test-set evaluation pipeline (nmr_infer over the full dataset) instead of the single-spectrum
  experimental workflow
- Molecules exceed the 19-heavy-atom limit for which the model was demonstrated and
  trained
- The goal is to predict stereochemistry — the model does not predict stereochemistry
  and this evaluation only scores canonical SMILES exact matches
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Framework providing the nmr_infer_single_spectrum entry point, preprocessing
    grids, and the released multitask model checkpoint used for experimental-spectra
    inference
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: Structure-assembly decoder of the multitask model that generates the 15 SMILES
    predictions per experimental spectrum
- name: Anaconda
  role: Package manager for installing the NMR2Struct environment (environment.yml,
    NMR_env) required to run inference
- name: RDKit
  role: Canonicalization of predicted and target SMILES strings so exact-match scoring
    can be performed on canonical forms
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:930ee64b7d439d9cba65b8b3a28d86974c662f041abeeed6aeda24260492be2e
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# simulated-versus-experimental-nmr-generalization-evaluation

## Summary

Benchmarks the NMR2Struct multitask spectrum-to-structure model on experimental 1D 1H/13C NMR spectra (106-spectrum set) and quantifies the drop in top-15 exact-match accuracy relative to the 69.6% baseline achieved on simulated spectra.

## When to use

Use when a model trained on simulated 1H and/or 13C NMR spectra must be evaluated on real experimental spectra, and the simulated-to-experimental accuracy gap needs to be quantified (e.g., top-15 exact-match rate ~33.0% vs 69.6% simulated).

## When NOT to use

- Input spectra are simulated MestreNova-predicted SpectraBase spectra — use the standard test-set evaluation pipeline (nmr_infer over the full dataset) instead of the single-spectrum experimental workflow
- Molecules exceed the 19-heavy-atom limit for which the model was demonstrated and trained
- The goal is to predict stereochemistry — the model does not predict stereochemistry and this evaluation only scores canonical SMILES exact matches

## Inputs

- 106 experimental 1D 1H and 13C NMR spectra (SI Section 5 of the authors' previous work) with known target molecules
- Pretrained multitask spectrum-to-structure model checkpoint from the released NMR2Struct model
- Shift-grid pickle files: example_configs/HNMR_shifts.p and example_configs/CNMR_shifts.p
- YAML configuration file with global_args, model, and inference sections
- 1H spectra as text/CSV files of (ppm, intensity) pairs and 13C spectra as comma-separated carbon ppm shifts

## Outputs

- 15 SMILES predictions per experimental spectrum from the multitask model
- Overall top-15 exact-match accuracy over the 106 experimental spectra (expected ~33.0%)
- Quantified simulated-to-experimental gap vs the 69.6% simulated baseline, recorded in experimental_eval_results.json

## How to apply

Load the 106 experimental 1H/13C NMR spectra and their known target molecules, then preprocess each with the NMR2Struct single-spectrum pipeline: interpolate the 1H spectrum onto the shift grid (example_configs/HNMR_shifts.p) with intensities normalized by the highest intensity via the --normalize flag, and bin the 13C shifts onto the grid (example_configs/CNMR_shifts.p). Run inference with the released multitask model through the nmr_infer_single_spectrum entry point, generating 15 SMILES predictions per input using the same top-k generation used in the paper. Score each spectrum by checking whether the target canonical SMILES appears within the set of valid predictions, following the analysis over processed_predictions.h5. Compute the overall exact-match accuracy over the 106 experimental spectra and record the gap from the 69.6% simulated-spectrum accuracy (expected ~33.0%) in experimental_eval_results.json.

## Related tools

- **NMR2Struct** (Framework providing the nmr_infer_single_spectrum entry point, preprocessing grids, and the released multitask model checkpoint used for experimental-spectra inference) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (Structure-assembly decoder of the multitask model that generates the 15 SMILES predictions per experimental spectrum)
- **Anaconda** (Package manager for installing the NMR2Struct environment (environment.yml, NMR_env) required to run inference)
- **RDKit** (Canonicalization of predicted and target SMILES strings so exact-match scoring can be performed on canonical forms)

## Examples

```
nmr_infer_single_spectrum --config config.yaml --hnmr_file HNMR_FILE --cnmr_file CNMR_FILE --hnmr_shifts example_configs/HNMR_shifts.p --cnmr_shifts example_configs/CNMR_shifts.p --ckpt CKPT --normalize
```

## Evaluation signals

- Computed exact-match accuracy over the 106 experimental spectra reproduces at approximately 33.0%
- The gap from the 69.6% simulated-spectrum top-15 accuracy is recorded in experimental_eval_results.json
- Each spectrum produces exactly 15 predictions in the same top-k manner used in the paper
- All predictions are canonicalized (RDKit) before target comparison, so scoring is on valid canonical SMILES
- Preprocessed spectra conform to the fixed grids: 1H interpolated onto the HNMR shift grid and 13C binned onto the CNMR shift grid

## Limitations

- The current model is trained on simulated spectra, which do not fully capture the nuances of real spectra — this is the source of the expected simulated-to-experimental performance drop
- The 106-spectrum benchmark is small, so the experimental accuracy estimate carries sampling uncertainty
- Scoring is exact-match only; structurally near-miss predictions are not credited in the headline metric
- Model limited to systems of up to 19 heavy atoms; molecules outside this range are out of scope

## Evidence

- [abstract] we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions, reducing the search space by up to 11 orders of magnitude.: "we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions"
- [discussion] simulated spectra miss real-world nuance: "that are not fully captured by the simulated spectra used to train the current model"
- [readme] single-spectrum inference entry point: "To infer the structures and/or substructures from a specific set of spectra, we recommend using the single spectrum inference entry point."
- [readme] --normalize flag description: [evidence span withheld — anchor status: not_found]
