---
name: hsqc-spectrum-candidate-ranking
description: Use when you have a query HSQC spectrum and a set of candidate structures
  sampled by MMST, and need to score/rank candidates to reproduce Figure 3d-style
  top-1 success rates per molecular-weight bin.
when_to_use_negative:
- The goal is generating new simulated NMR/IR training data (use the 2.0/3.0 data-generation
  notebooks with SGNN or Chemprop-IR instead)
- You need to train or fine-tune MMST rather than evaluate out-of-distribution inference
  performance
- The query spectra are not HSQC-based (MMST also consumes NMR, COSY, IR, and MS modalities;
  this skill targets the HSQC-ranked PubChem benchmark)
tools:
- name: MultiModalSpectralTransformer
  role: Transformer model run in multinomial-sampling mode to generate candidate structures,
    whose outputs are ranked against the query HSQC spectrum
  repo: mpriessner/MultiModalSpectralTransformer
- name: PyTorch
  role: Backend for running MMST model inference and sampling under CUDA 11.1 GPU
    acceleration
- name: RDKit
  role: Handles molecular structures/SMILES when comparing sampled candidate structures
    to the correct answer
- name: Jupyter Lab
  role: Runs the experiment notebook (1.0_Experiment_Notebook.ipynb) that reproduces
    the PubChem benchmark experiments
- name: Conda
  role: Provides the pinned Python 3.7.12 NMR_Structure_Elucidator environment in
    which the ranking workflow runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ange.202517611
    title: Advancing Structure Elucidation with a Flexible Multi‐Spectral AI Model
  source_content_hash: sha256:5c0b178515773181aff64ccdc63e2784076a199588eec3bd04661f5702756242
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mpriessner/MultiModalSpectralTransformer
  tools:
  - CUDA
  - Conda
  - Jupyter Lab
  - MultiModalSpectralTransformer
  - PyTorch
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# hsqc-spectrum-candidate-ranking

## Summary

Ranks MMST-sampled candidate molecular structures against a query HSQC spectrum and computes top-1 success rates, used to benchmark out-of-distribution performance on the PubChem molecular-weight test sets (0–250, 250–350, 350–500 Da).

## When to use

Use when you have a query HSQC spectrum and a set of candidate structures sampled by MMST, and need to score/rank candidates to reproduce Figure 3d-style top-1 success rates per molecular-weight bin.

## When NOT to use

- The goal is generating new simulated NMR/IR training data (use the 2.0/3.0 data-generation notebooks with SGNN or Chemprop-IR instead)
- You need to train or fine-tune MMST rather than evaluate out-of-distribution inference performance
- The query spectra are not HSQC-based (MMST also consumes NMR, COSY, IR, and MS modalities; this skill targets the HSQC-ranked PubChem benchmark)

## Inputs

- Query HSQC spectra for PubChem test molecules (three MW bins: 0–250, 250–350, 350–500 Da, 1000 molecules each)
- Pre-trained ZINC-trained MMST model checkpoints (models directory, from Zenodo 10.5281/zenodo.14712886)
- PubChem test sets in the data/PubChem_dataset directory (downloaded from the Zenodo data archives)
- Correct/reference structures (SMILES) per test molecule for top-1 verification

## Outputs

- Ranked list of candidate structures per test molecule (correct structure position)
- top1_success_rates.csv with one top-1 success rate per molecular-weight bin

## How to apply

Activate the NMR_Structure_Elucidator conda environment (Python 3.7.12, PyTorch with CUDA 11.1 GPU acceleration) and load the pre-trained ZINC-trained MMST models from the Zenodo deposit (10.5281/zenodo.14712886 / 16076914). Load the three PubChem test sets (0–250, 250–350, 350–500 Da bins, 1000 molecules each) from the data/PubChem_dataset directory. For each test molecule, run MMST structure prediction with multinomial sampling to generate a set of candidate structures, then rank the sampled candidates against the molecule's query HSQC spectrum. Compute the top-1 success rate per molecular-weight bin as the fraction of molecules whose correct structure is ranked first, write the three rates to top1_success_rates.csv, and compare against the reported Figure 3d values (39% / 43% / 13%, with set 2 performing best).

## Related tools

- **MultiModalSpectralTransformer** (Transformer model run in multinomial-sampling mode to generate candidate structures, whose outputs are ranked against the query HSQC spectrum) — mpriessner/MultiModalSpectralTransformer
- **PyTorch** (Backend for running MMST model inference and sampling under CUDA 11.1 GPU acceleration)
- **RDKit** (Handles molecular structures/SMILES when comparing sampled candidate structures to the correct answer)
- **Jupyter Lab** (Runs the experiment notebook (1.0_Experiment_Notebook.ipynb) that reproduces the PubChem benchmark experiments)
- **Conda** (Provides the pinned Python 3.7.12 NMR_Structure_Elucidator environment in which the ranking workflow runs)

## Examples

```
conda activate NMR_Structure_Elucidator && jupyter lab 1.0_Experiment_Notebook.ipynb
```

## Evaluation signals

- The three per-bin top-1 success rates in top1_success_rates.csv match the Figure 3d values: 39% (0–250 Da), 43% (250–350 Da), 13% (350–500 Da)
- Set 2 (250–350 Da) has the highest top-1 rate of the three bins
- Each binned test set contains 1000 molecules, so success rates are fractions over 1000 top-1 rankings
- In-distribution reference: the ZINC-trained model attains 72% top-1 / 80% top-3 accuracy, so large deviations on ZINC_4000 indicate a setup error
- Environment check passes: the NMR_Structure_Elucidator conda environment (Python 3.7.12, CUDA 11.1) runs one of the simpler notebooks without module or model-loading errors

## Limitations

- Out-of-distribution degradation is severe for heavier compounds: only 13% top-1 on the 350–500 Da set versus 43% on 250–350 Da, reflecting the applicability domain of the ZINC-trained model
- MMST is trained solely on simulated spectral data; performance on experimental spectra, while good, is not identical to the simulated-training regime
- Strict environment pins are required: Python 3.7.12 and CUDA 11.1; mismatches cause CUDA kernel-image errors, ModuleNotFoundError, or ModuleAttributeError on model loading
- Data/model archives from Zenodo must be extracted into the exact folder layout (data/PubChem_dataset, models/mmst, etc.) or FileNotFoundError will occur

## Evidence

- [abstract] MMST in-sample accuracy: "Trained on 4 million simulated compounds, MMST achieves 72% and 80% as top-1 and top-3 accuracy, respectively."
- [readme] MMST modalities: "MultiModalSpectralTransformer is a transformer-based architecture that integrates various spectroscopic modalities (NMR, HSQC, COSY, IR) for automated molecular structure prediction"
- [methods] pinned environment: "conda create -y -c conda-forge -n NMR_Structure_Elucidator python=3.7.12"
- [readme] experiment notebook purpose: "This notebook is used to reproduce the experiments."
- [methods] pretrained models source: "download the necessary data and pre-trained models from our Zenodo repository: [https://doi.org/10.5281/zenodo.14712886]"
