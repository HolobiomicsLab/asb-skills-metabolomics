---
name: large-scale-benchmark-evaluation-over-500k-compounds
description: 'Use when reproducing or extending the paper''s benchmark evaluation:
  MMST inference with greedy sampling over the >490,000 held-out ZINC test compounds,
  scoring generated SMILES with RDKit, and reproducing the reported ablation metrics.'
when_to_use_negative:
- Input is experimental spectra from a small target sample rather than the large simulated
  ZINC benchmark — use the experimental-spectra evaluation workflow instead
- Goal is generating new simulated spectral training data (SGNN / Chemprop-IR notebooks)
  rather than evaluating an existing model checkpoint
- Only the ZINC_4000 or test_data subsets are available — the >490,000-compound scale
  and the reported ablation statistics require the full ZINC_dataset
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MultiModalSpectralTransformer
  role: Transformer model run in greedy-sampling inference mode on the ZINC test set,
    once per modality-ablation condition
  repo: https://github.com/your-username/MultiModalSpectralTransformer.git
- name: RDKit
  role: Scores each generated SMILES for validity, exact-match identity, and Tanimoto
    similarity against the reference molecule
- name: Python 3.7.12
  role: Runtime for the evaluation notebooks (tested with Python 3.7.12)
- name: PyTorch
  role: Deep-learning framework executing MMST inference with CUDA 11.1 GPU acceleration
- name: Conda
  role: Creates the NMR_Structure_Elucidator environment (python=3.7.12) required
    to run the evaluation
- name: Jupyter Lab
  role: Environment used to run 1.0_Experiment_Notebook.ipynb, which reproduces the
    paper's evaluation experiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ange.202517611
    title: Advancing Structure Elucidation with a Flexible Multi‐Spectral AI Model
  source_content_hash: sha256:3b1e2005020c3a9ea7b61f561450fe8084df25633114e46369cdded33d48a2ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/your-username/MultiModalSpectralTransformer.git
  tools:
  - CUDA
  - Conda
  - Jupyter Lab
  - MultiModalSpectralTransformer
  - PyTorch
  - Python
  - Python 3.7.12
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# large-scale-benchmark-evaluation-over-500k-compounds

## Summary

Reproduces large-scale benchmark evaluation of the MultiModalSpectralTransformer (MMST) structure-elucidation model over >490,000 held-out ZINC compounds, including modality ablations (all-modalities vs no-HSQC vs no-COSY) scored with RDKit metrics (exact match, invalid-SMILES rate, Tanimoto similarity, correct-SMILES probability).

## When to use

Use when reproducing or extending the paper's benchmark evaluation: MMST inference with greedy sampling over the >490,000 held-out ZINC test compounds, scoring generated SMILES with RDKit, and reproducing the reported ablation metrics.

## When NOT to use

- Input is experimental spectra from a small target sample rather than the large simulated ZINC benchmark — use the experimental-spectra evaluation workflow instead
- Goal is generating new simulated spectral training data (SGNN / Chemprop-IR notebooks) rather than evaluating an existing model checkpoint
- Only the ZINC_4000 or test_data subsets are available — the >490,000-compound scale and the reported ablation statistics require the full ZINC_dataset

## Inputs

- >490,000 held-out ZINC test compounds from the ZINC_dataset directory
- Pre-trained MultiModalSpectralTransformer checkpoints extracted into the models directory (Zenodo 10.5281/zenodo.14712886)
- Spectral input tensors (HSQC, COSY, IR, 1H/13C NMR modalities) for each ZINC compound
- Reference SMILES strings for each test compound

## Outputs

- modality_ablation_results.csv with per-condition exact-match accuracy, invalid-SMILES rate, mean Tanimoto similarity, and correct-SMILES rate
- Per-compound generated candidate SMILES scored with RDKit

## How to apply

Load the held-out ZINC test compounds from the ZINC_dataset directory and the pre-trained MMST checkpoints extracted into the models directory (downloaded from Zenodo 10.5281/zenodo.14712886). Activate the NMR_Structure_Elucidator conda environment (Python 3.7.12, PyTorch with CUDA 11.1) and run 1.0_Experiment_Notebook.ipynb in Jupyter Lab. Run MMST inference with greedy sampling on all >490,000 compounds using the full all-modality input, then repeat with each spectral modality (HSQC, COSY, IR, 1H/13C NMR) individually omitted. Score every generated candidate with RDKit: exact-match identity against reference SMILES, validity of generated SMILES, and Tanimoto similarity between predicted and reference molecules. Aggregate exact-match accuracy, invalid-SMILES rate, mean Tanimoto, and correct-SMILES rate per condition, and check reproduction against the reported values: exact match 59.7% (all modalities) vs 7.7% (no HSQC) and 8.1% (no COSY); invalid rates 2.0/9.2/7.9%; Tanimoto 0.82/0.43/0.48; correct-SMILES 0.51/0.04/0.06. Write the per-condition table to modality_ablation_results.csv.

## Related tools

- **MultiModalSpectralTransformer** (Transformer model run in greedy-sampling inference mode on the ZINC test set, once per modality-ablation condition) — https://github.com/your-username/MultiModalSpectralTransformer.git
- **RDKit** (Scores each generated SMILES for validity, exact-match identity, and Tanimoto similarity against the reference molecule)
- **Python 3.7.12** (Runtime for the evaluation notebooks (tested with Python 3.7.12))
- **PyTorch** (Deep-learning framework executing MMST inference with CUDA 11.1 GPU acceleration)
- **Conda** (Creates the NMR_Structure_Elucidator environment (python=3.7.12) required to run the evaluation)
- **Jupyter Lab** (Environment used to run 1.0_Experiment_Notebook.ipynb, which reproduces the paper's evaluation experiments)

## Examples

```
conda activate NMR_Structure_Elucidator && jupyter lab  # open 1.0_Experiment_Notebook.ipynb to run MMST greedy-sampling inference on ZINC_dataset and write modality_ablation_results.csv
```

## Evaluation signals

- All-modality exact-match accuracy reproduces 59.7% on the >490,000 ZINC compounds within reproducibility tolerance
- No-HSQC and no-COSY ablations reproduce 7.7% and 8.1% exact match, confirming 2D NMR as the most critical input
- Invalid-SMILES rates match reported values 2.0% (all modalities), 9.2% (no HSQC), 7.9% (no COSY)
- Mean Tanimoto similarity matches reported 0.82 / 0.43 / 0.48 and correct-SMILES rates match 0.51 / 0.04 / 0.06 across conditions
- modality_ablation_results.csv contains one aggregated row per ablation condition with all four metrics populated

## Limitations

- Requires a high-performance NVIDIA GPU with CUDA 11.1 support (e.g., V100 or K80), at least 16GB RAM, and ~50GB storage for datasets, checkpoints, and results
- Model must be trained/fine-tuned on simulated data; the model was trained on 4 million simulated compounds, so out-of-distribution chemical spaces may degrade performance (the paper addresses this via an active learning improvement cycle)
- Missing or misplaced Zenodo archives cause FileNotFoundError or ModuleAttributeError at model-load time; all data.tar.xz.part*, models.tar.xz.part*, and past_experiments.tar.xz.part* parts must be extracted to the correct directories
- Details on the full usage and improvement cycle workflow are deferred to Section 3 of the Electronic Supplementary Information

## Evidence

- [results] exact match accuracy dropped from 59.7% with all modalities to 7.7% without HSQC and 8.1% without COSY, identifying 2D NMR data as the most critical input: [evidence span withheld — anchor status: not_found]
- [readme] reproduce the paper's evaluation experiments: "1.0_Experiment_Notebook.ipynb
   - This notebook is used to reproduce the experiments."
- [readme] Zenodo models download: "download the necessary data and pre-trained models from our Zenodo repository"
- [readme] environment setup: "conda create -y -c conda-forge -n NMR_Structure_Elucidator python=3.7.12"
