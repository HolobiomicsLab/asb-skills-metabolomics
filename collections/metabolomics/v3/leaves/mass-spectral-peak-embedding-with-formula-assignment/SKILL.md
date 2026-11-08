---
name: mass-spectral-peak-embedding-with-formula-assignment
description: Use when you have tandem mass spectra (e.g., MassSpecGym or NPLIB1/CANOPUS-format
  entries) with a known or inferrable chemical formula and need de novo candidate
  molecular structures ranked by sampling frequency.
when_to_use_negative:
- The chemical formula cannot be assigned from the spectrum (no high-resolution MS1/isotopic-trace
  support), since generation is formula-restricted by construction
- The goal is library/database annotation of existing reference spectra rather than
  de novo generation of structures absent from PubChem or HMDB
- The input is already a fingerprint or molecular structure table with no mass spectra
  — use fp2mol pretraining-style training instead of spectrum-conditioned inference
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DiffMS
  role: Formula-restricted encoder-decoder diffusion model that generates molecules
    conditioned on mass spectra; end-to-end checkpoints from Zenodo record 23542112
  repo: https://github.com/coleygroup/DiffMS
- name: MIST
  role: Formula transformer used as the spectrum encoder predicting 4,096-bit Morgan
    fingerprints; encoder checkpoints from Zenodo record 23303289
  repo: https://github.com/coleygroup/MIST
- name: DiGress
  role: Discrete denoising graph-diffusion codebase from which the DiffMS decoder
    and codebase are adapted
  repo: https://github.com/cvignac/DiGress
- name: RDKit 2024.09.4
  role: Validity and connectivity checks on sampled molecules; Morgan fingerprint
    Tanimoto similarity scoring
- name: SIRIUS
  role: Tool for inferring chemical formulae from high-resolution MS1 data and isotopic
    traces when formulas are not provided
- name: PyTorch 2.3.1
  role: Training/inference backend for the encoder and diffusion decoder (with torch_geometric
    2.3.1, cuda 11.8)
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:8eda0fdfa5a5eb475afa8f193ff5162703be6c3cd537ea47e235c0a8ee190353
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiGress
  - DiffMS
  - MIST
  - PyTorch
  - PyTorch 2.3.1
  - RDKit 2024.09.4
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# mass-spectral-peak-embedding-with-formula-assignment

## Summary

Encodes tandem mass spectra into latent embeddings and formula-constrained molecular generations by combining a spectrum-to-fingerprint encoder (MIST) with a formula-restricted discrete graph diffusion decoder (DiffMS), pretrained on 2.8M fingerprint-structure pairs.

## When to use

Use when you have tandem mass spectra (e.g., MassSpecGym or NPLIB1/CANOPUS-format entries) with a known or inferrable chemical formula and need de novo candidate molecular structures ranked by sampling frequency.

## When NOT to use

- The chemical formula cannot be assigned from the spectrum (no high-resolution MS1/isotopic-trace support), since generation is formula-restricted by construction
- The goal is library/database annotation of existing reference spectra rather than de novo generation of structures absent from PubChem or HMDB
- The input is already a fingerprint or molecular structure table with no mass spectra — use fp2mol pretraining-style training instead of spectrum-conditioned inference

## Inputs

- Tandem mass spectra in MassSpecGym or NPLIB1 (CANOPUS/GNPS-derived) format, processed via data_processing/ scripts
- Chemical formula per spectrum (inferred from high-resolution MS1 data and isotopic traces, or provided)
- Fingerprint-structure pretraining pairs (2.8M, sampled from DSSTox, HMDB, COCONUT, MOSES)
- Pretrained checkpoints: DiffMS end-to-end models from zenodo.org/records/23542112 and MIST encoder checkpoints from zenodo.org/records/23303289

## Outputs

- 100 sampled molecular structures (graphs/SMILES) per spectrum, restricted to the given formula's heavy-atom composition
- Ranked top-k (k=1, 10) candidate molecules per spectrum after invalid/disconnected filtering
- Predicted 4,096-bit Morgan fingerprints (radius 2, without chirality) from the spectrum encoder
- Benchmark metrics: top-k accuracy, top-k maximum Tanimoto similarity (Morgan fingerprints), top-k minimum MCES distance

## How to apply

Set up the environment with PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1, and RDKit 2024.09.4 (conda env 'diffms'). Download and process the finetuning and pretraining data by running the data_processing/ scripts 00-03 in order (00_download_fp2mol_data.sh through 03_preprocess_fp2mol.sh). Treat the chemical formula as given (inferred from high-resolution MS1 data and isotopic traces via SIRIUS or the MIST formula transformer), then pretrain the spectrum encoder to predict 4,096-bit Morgan fingerprints (radius 2, without chirality) and the diffusion decoder on fingerprint-structure pairs (2.8M from DSSTox/HMDB/COCONUT/MOSES) before end-to-end finetuning via spec2mol_main.py with the dataset set to 'msg' (MassSpecGym) or 'canopus' (NPLIB1) in configs/config.yaml. At inference, sample 100 molecules per spectrum with the graph diffusion decoder conditioned on the spectrum and formula-derived atom types, remove invalid or disconnected molecules with RDKit, and rank top-k (k=1, 10) candidates by frequency.

## Related tools

- **DiffMS** (Formula-restricted encoder-decoder diffusion model that generates molecules conditioned on mass spectra; end-to-end checkpoints from Zenodo record 23542112) — https://github.com/coleygroup/DiffMS
- **MIST** (Formula transformer used as the spectrum encoder predicting 4,096-bit Morgan fingerprints; encoder checkpoints from Zenodo record 23303289) — https://github.com/coleygroup/MIST
- **DiGress** (Discrete denoising graph-diffusion codebase from which the DiffMS decoder and codebase are adapted) — https://github.com/cvignac/DiGress
- **RDKit 2024.09.4** (Validity and connectivity checks on sampled molecules; Morgan fingerprint Tanimoto similarity scoring)
- **SIRIUS** (Tool for inferring chemical formulae from high-resolution MS1 data and isotopic traces when formulas are not provided)
- **PyTorch 2.3.1** (Training/inference backend for the encoder and diffusion decoder (with torch_geometric 2.3.1, cuda 11.8))

## Examples

```
conda create -y -c conda-forge -n diffms rdkit=2024.09.4 python=3.9; python src/spec2mol_main.py  # config.yaml dataset='msg' or 'canopus'
```

## Evaluation signals

- Reproduced NPLIB1 top-1 accuracy should match the reported 8.34% (MCES 11.95, Tanimoto 0.35) with encoder pretraining, within stochastic-sampling tolerance
- Reproduced MassSpecGym top-1 accuracy should match the reported 2.30% (Tanimoto 0.28)
- Ablation check: without encoder pretraining, NPLIB1 top-1 accuracy should drop to ~4.36% (MCES 12.34, Tanimoto 0.31)
- Every sampled molecule passes RDKit validity and connectivity checks and its heavy-atom composition matches the input formula
- Checkpoint files load successfully via general.load_weights in configs/general/general_default.yaml from the Zenodo records

## Limitations

- De novo generation of the exact molecular structure remains challenging; outputs are often structurally close matches rather than the exact molecule
- Performance depends strongly on encoder pretraining — omitting it halves NPLIB1 top-1 accuracy (8.34% -> 4.36%)
- Evaluation relies on stochastic sampling of 100 molecules per spectrum, so reproduced metrics vary run-to-run
- The codebase is adapted from DiGress, which has a known bug where dim_ffy is ignored in GraphTransformer construction (default 2048 used); graph_tool and torch_geometric conflict on MacOS

## Evidence

- [methods] encoder pretraining: [evidence span withheld — anchor status: not_found]
- [methods] sampling and ranking: [evidence span withheld — anchor status: not_found]
- [readme] MIST fingerprint outputs: [evidence span withheld — anchor status: external_doc]
