---
name: multimodal-transformer-fine-tuning
description: Use when a pre-trained MMST model fails to correctly predict structures
  for a set of molecules (out-of-distribution chemical space, e.g. PubChem sets) and
  analogues plus simulated spectra can be generated to recover predictions.
when_to_use_negative:
- The molecule set is already in-distribution and the pre-trained MMST achieves the
  expected 72% top-1 accuracy without fine-tuning
- No spectral simulation pipeline (SGNN / Chemprop-IR) or GPU resources are available
  to generate training data for analogues
- The task is training MMST from scratch on a new dataset (e.g. the Alberts benchmark)
  rather than fine-tuning a pre-trained checkpoint
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3332
tools:
- name: MultiModalSpectralTransformer
  role: Transformer model fine-tuned on newly simulated multi-spectral (NMR, HSQC,
    COSY, IR) data and re-run for structure prediction on failed molecules
  repo: https://github.com/your-username/MultiModalSpectralTransformer.git
- name: Mol2Mol
  role: Generates 30 analogues for each failed molecule via the deep-molecular-optimization
    module
- name: SGNN
  role: Simulates NMR spectra for the generated analogues (2.0_NMR_Data_Generation.ipynb)
- name: Chemprop-IR
  role: Simulates IR spectra for the generated analogues (3.0_Chemprop_IR_Data_Generation.ipynb)
- name: PyTorch
  role: Deep learning framework for training/fine-tuning MMST under CUDA 11.1 GPU
    acceleration
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/ange.202517611
    title: Advancing Structure Elucidation with a Flexible Multi‐Spectral AI Model
  source_content_hash: sha256:af3e2c3d93bee315013758565863ac4ec81c1b2e2129c475f457467e4ec5e6ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/your-username/MultiModalSpectralTransformer.git
  tools:
  - CUDA
  - Chemprop-IR
  - Conda
  - Jupyter Lab
  - Mol2Mol
  - MultiModalSpectralTransformer
  - PyTorch
  - Python
  - RDKit
  - SGNN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# multimodal-transformer-fine-tuning

## Summary

Fine-tune the MultiModalSpectralTransformer (MMST) on newly simulated multi-spectral data (NMR, HSQC, COSY, IR) generated for out-of-distribution molecules, as part of the active learning improvement cycle that adapts the model to new chemical spaces.

## When to use

Use when a pre-trained MMST model fails to correctly predict structures for a set of molecules (out-of-distribution chemical space, e.g. PubChem sets) and analogues plus simulated spectra can be generated to recover predictions.

## When NOT to use

- The molecule set is already in-distribution and the pre-trained MMST achieves the expected 72% top-1 accuracy without fine-tuning
- No spectral simulation pipeline (SGNN / Chemprop-IR) or GPU resources are available to generate training data for analogues
- The task is training MMST from scratch on a new dataset (e.g. the Alberts benchmark) rather than fine-tuning a pre-trained checkpoint

## Inputs

- 100 initially failed molecules per dataset from the PubChem_dataset directory
- Pre-trained models from Zenodo (mol2mol, mmst, sgnn, chemprop-ir)
- Newly simulated multi-spectral data (NMR via SGNN, IR via Chemprop-IR) for generated analogues

## Outputs

- Fine-tuned MultiModalSpectralTransformer model checkpoint
- improvement_cycle_results.csv with post-improvement-cycle top-1 success rates and failure rates per PubChem set
- Top-1 success rates recovered into the 57-72% range for previously failed molecules

## How to apply

Load the 100 failed molecules per dataset (e.g. from the PubChem_dataset directory) and the pre-trained models (mol2mol, mmst, sgnn, chemprop-ir) from the Zenodo repository. Generate 30 analogues per failed molecule with Mol2Mol from the deep-molecular-optimization module, then simulate NMR spectra with SGNN (2.0_NMR_Data_Generation.ipynb) and IR spectra with Chemprop-IR (3.0_Chemprop_IR_Data_Generation.ipynb). Fine-tune the MMST on this newly simulated multi-spectral data following the improvement cycle workflow, then re-run structure prediction on the original failed molecules via 1.0_Experiment_Notebook.ipynb and compute post-improvement-cycle top-1 success rates. Success is judged by recovered top-1 rates in the reported 57-72% range with failure rates at or below 1%.

## Related tools

- **MultiModalSpectralTransformer** (Transformer model fine-tuned on newly simulated multi-spectral (NMR, HSQC, COSY, IR) data and re-run for structure prediction on failed molecules) — https://github.com/your-username/MultiModalSpectralTransformer.git
- **Mol2Mol** (Generates 30 analogues for each failed molecule via the deep-molecular-optimization module)
- **SGNN** (Simulates NMR spectra for the generated analogues (2.0_NMR_Data_Generation.ipynb))
- **Chemprop-IR** (Simulates IR spectra for the generated analogues (3.0_Chemprop_IR_Data_Generation.ipynb))
- **PyTorch** (Deep learning framework for training/fine-tuning MMST under CUDA 11.1 GPU acceleration)

## Examples

```
jupyter lab  # then run 2.0_NMR_Data_Generation.ipynb and 3.0_Chemprop_IR_Data_Generation.ipynb, fine-tune mmst, and evaluate via 1.0_Experiment_Notebook.ipynb
```

## Evaluation signals

- Post-improvement-cycle top-1 success rates fall within the reported 57-72% range across the PubChem molecular-weight sets
- Failure rates after the improvement cycle are at or below 1% per PubChem set
- improvement_cycle_results.csv is written with per-set top-1 success and failure rates
- Pre-trained model checkpoints load without torch.nn.modules.module.ModuleAttributeError, confirming correct Zenodo model files and format

## Limitations

- Requires an NVIDIA GPU with CUDA 11.1 support (e.g. NVIDIA V100 or K80), at least 16GB RAM and 50GB storage
- The full improvement cycle workflow and HTML GUI usage are deferred to Section 3 of the Electronic Supplementary Information (ESI)
- MMST is trained solely on simulated data; performance on experimental spectra may differ, and CUDA/GPU version mismatches cause 'no kernel image is available' errors

## Evidence

- [results] Applying the improvement cycle to 100 failed molecules from each PubChem set recovered the predictions, yielding top-1 success rates of 57% to 72% across the sets: [evidence span withheld — anchor status: not_found]
