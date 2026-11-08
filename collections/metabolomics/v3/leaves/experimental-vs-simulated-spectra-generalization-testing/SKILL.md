---
name: experimental-vs-simulated-spectra-generalization-testing
description: Use when a model trained solely on simulated spectra (SGNN NMR, Chemprop-IR)
  must be evaluated on experimental spectral data to measure out-of-distribution generalization
  and improvement-cycle recovery.
when_to_use_negative:
- The goal is training or fine-tuning the model on new data rather than evaluating
  out-of-distribution generalization of existing pre-trained checkpoints
- The compound benchmark differs from the curated 34-compound validation set with
  experimental spectra, so results cannot be compared against the reported reference
  accuracies
- No improvement-cycle models (mol2mol/deep-molecular-optimization checkpoints) or
  experimental spectra are available, making only the simulated condition runnable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3473
tools:
- name: MultiModalSpectralTransformer
  role: Transformer model under test that predicts structures from NMR/HSQC/COSY/IR/MS
    spectral modalities; baseline inference is run on both simulated and experimental
    inputs
  repo: https://github.com/mpriessner/MultiModalSpectralTransformer
- name: SGNN
  role: Generates simulated NMR spectra for the simulated-condition baseline (notebook
    2.0_NMR_Data_Generation.ipynb)
- name: Chemprop-IR
  role: Produces simulated IR spectra for the simulated-condition baseline (notebook
    3.0_Chemprop_IR_Data_Generation.ipynb)
- name: Jupyter Lab
  role: Environment in which 1.0_Experiment_Notebook.ipynb executes the baseline and
    improvement-cycle evaluations
- name: PyTorch
  role: Deep-learning backend for MMST inference under CUDA 11.1 GPU acceleration
- name: Conda
  role: Creates the NMR_Structure_Elucidator environment (Python 3.7.12) with installs.sh
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ange.202517611
    title: Advancing Structure Elucidation with a Flexible Multi‐Spectral AI Model
  source_content_hash: sha256:061d9fa3c328fa9c3323c7f31acb1b6e719ce047b9ee9d9aff697a9fdf726ec9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mpriessner/MultiModalSpectralTransformer
  tools:
  - CUDA
  - Chemprop-IR
  - Conda
  - Jupyter Lab
  - MultiModalSpectralTransformer
  - PyTorch
  - Python
  - RDKit
  - SGNN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# experimental-vs-simulated-spectra-generalization-testing

## Summary

Benchmark a multi-spectral structure-prediction model (MMST) on a curated 34-compound set under both simulated (SGNN NMR, Chemprop-IR) and experimental 1H/13C/HSQC/COSY/IR/MS spectral inputs, then quantify how much the active-learning improvement cycle (IC) recovers accuracy lost to the simulated-to-experimental domain shift.

## When to use

Use when a model trained solely on simulated spectra (SGNN NMR, Chemprop-IR) must be evaluated on experimental spectral data to measure out-of-distribution generalization and improvement-cycle recovery.

## When NOT to use

- The goal is training or fine-tuning the model on new data rather than evaluating out-of-distribution generalization of existing pre-trained checkpoints
- The compound benchmark differs from the curated 34-compound validation set with experimental spectra, so results cannot be compared against the reported reference accuracies
- No improvement-cycle models (mol2mol/deep-molecular-optimization checkpoints) or experimental spectra are available, making only the simulated condition runnable

## Inputs

- Experimental 1H/13C/HSQC/COSY/IR/MS spectra for the 34 validation compounds (from the Zenodo data archives)
- Simulated spectral inputs: SGNN-generated NMR and Chemprop-IR spectra for the same 34 compounds
- Pre-trained models (mmst, sgnn, chemprop-ir, mol2mol) extracted into models/
- 1.0_Experiment_Notebook.ipynb from past_experiments/

## Outputs

- Top-1, top-3, and top-10 accuracy per condition (baseline MMST and improvement cycle, simulated vs experimental inputs)
- Exported results table: figure6a_topk_accuracy.csv
- Reported reference comparison values (simulated 72%/80% top-1/top-3; experimental baseline 0/3/81% IC-on-target 31/56/81%; IC-on-analogue 12/38%)

## How to apply

Extract the Zenodo archives (data.tar.xz, models.tar.xz, past_experiments.tar.xz) into data/, models/, and past_experiments/ of the MultiModalSpectralTransformer repo, create the NMR_Structure_Elucidator conda env (Python 3.7.12, PyTorch with CUDA 11.1), and open 1.0_Experiment_Notebook.ipynb in Jupyter Lab. Run baseline MMST inference over the 34-compound benchmark twice: once with experimental 1H/13C/HSQC/COSY/IR/MS spectra deposited in the data directory, and once with simulated inputs (SGNN-generated NMR plus Chemprop-IR spectra). Then run the improvement-cycle evaluation twice — once starting from the on-target molecule and once from a regioisomer analogue — recording top-k predictions. Aggregate top-1, top-3, and top-10 accuracy per condition (baseline vs IC, simulated vs experimental) and export as figure6a_topk_accuracy.csv. The rationale is that simulated-only training degrades sharply on real spectra (reported 0% top-1/top-3 experimentally vs 72% top-1 simulated), so measuring the delta and the IC's recovery without fine-tuning on experimental data is the key decision point.

## Related tools

- **MultiModalSpectralTransformer** (Transformer model under test that predicts structures from NMR/HSQC/COSY/IR/MS spectral modalities; baseline inference is run on both simulated and experimental inputs) — https://github.com/mpriessner/MultiModalSpectralTransformer
- **SGNN** (Generates simulated NMR spectra for the simulated-condition baseline (notebook 2.0_NMR_Data_Generation.ipynb))
- **Chemprop-IR** (Produces simulated IR spectra for the simulated-condition baseline (notebook 3.0_Chemprop_IR_Data_Generation.ipynb))
- **Jupyter Lab** (Environment in which 1.0_Experiment_Notebook.ipynb executes the baseline and improvement-cycle evaluations)
- **PyTorch** (Deep-learning backend for MMST inference under CUDA 11.1 GPU acceleration)
- **Conda** (Creates the NMR_Structure_Elucidator environment (Python 3.7.12) with installs.sh)

## Examples

```
conda create -y -c conda-forge -n NMR_Structure_Elucidator python=3.7.12 && conda activate NMR_Structure_Elucidator && bash installs.sh && jupyter lab
```

## Evaluation signals

- All top-1, top-3, and top-10 accuracy values lie within the 0–100% range
- figure6a_topk_accuracy.csv is present and well-formed after aggregation
- Reproduced simulated-condition accuracies match the reported reference values (72% top-1, 80% top-3)
- Experimental baseline shows the expected near-zero performance (0% top-1/top-3, 3% top-10) confirming the domain-shift measurement is meaningful
- Improvement-cycle on-target recovery reaches approximately 31% top-1 / 56% top-3 / 81% top-10 on experimental spectra without experimental fine-tuning

## Limitations

- The model requires significant computational resources: an NVIDIA GPU with CUDA 11.1 (e.g. V100 or K80), at least 16GB RAM, and at least 50GB storage
- Results are only comparable to the published reference values when using the exact 34-compound set and deposited experimental spectra from Zenodo
- Model-loading errors ('ModuleAttributeError') occur if the downloaded Zenodo model files do not match the expected format; CUDA kernel-image errors occur if the CUDA version differs from 11.1
- Detailed improvement-cycle usage instructions are deferred to Electronic Supplementary Information Section 3 rather than the README

## Evidence

- [abstract] Notably, despite training solely on simulated data, MMST demonstrates good performance with experimental spectra.: "Notably, despite training solely on simulated data, MMST demonstrates good performance with experimental spectra."
- [abstract] Trained on 4 million simulated compounds, MMST achieves 72% and 80% as top-1 and top-3 accuracy, respectively.: "Trained on 4 million simulated compounds, MMST achieves 72% and 80% as top-1 and top-3 accuracy, respectively."
- [readme] This notebook is used to reproduce the experiments.: "1. **1.0_Experiment_Notebook.ipynb**
   - This notebook is used to reproduce the experiments."
- [readme] This notebook is used to generate simulated NMR data using the SGNN network.: "2. **2.0_NMR_Data_Generation.ipynb**
   - This notebook is used to generate simulated NMR data using the SGNN network."
- [readme] Python 3.7.x is required (tested with Python 3.7.12).: "- **Python**: Python 3.7.x is required (tested with Python 3.7.12)."
- [readme] CUDA 11.1 is required for the GPU acceleration (required for PyTorch).: "- **CUDA**: CUDA 11.1 is required for the GPU acceleration (required for PyTorch)."
- [readme] Extract data archives command from README setup instructions: "cat data.tar.xz.part* > data.tar.xz
   tar -xJf data.tar.xz -C data/"
