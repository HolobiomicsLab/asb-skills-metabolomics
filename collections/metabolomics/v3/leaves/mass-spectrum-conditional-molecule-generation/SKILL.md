---
name: mass-spectrum-conditional-molecule-generation
description: Use when you have a tandem mass spectrum plus an inferred molecular formula
  and need candidate molecular structures ranked by top-k accuracy, MCES distance,
  or Tanimoto similarity against a ground-truth molecule.
when_to_use_negative:
- You only need candidate molecular formulas rather than full structures — formula-restricted
  tools like SIRIUS, BUDDY, or MIST-CF address that upstream step
- The target structures exist in standard virtual chemical libraries (PubChem, Human
  Metabolome Database) where library lookup suffices
- You lack a reliably inferred molecular formula, since DiffMS is formula-restricted
  (heavy-atom-constrained) and takes the formula as given
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3332
tools:
- name: DiffMS
  role: Formula-restricted encoder-decoder diffusion model that generates molecular
    graphs conditioned on mass spectra
  repo: github.com/coleygroup/DiffMS
- name: MIST
  role: Formula transformer used as the spectrum encoder, pretrained to predict 4,096-bit
    Morgan fingerprints (radius 2, no chirality) from spectra
  repo: coleygroup/MIST
- name: SIRIUS
  role: Infers molecular formulas from high-resolution MS1 data and isotopic traces
    to condition the formula-restricted generation
- name: PyTorch
  role: Deep-learning framework for training and running the diffusion model (tested
    with PyTorch 2.3.1, cuda 11.8)
- name: torch_geometric
  role: Graph library backend for the molecular-graph diffusion model (tested with
    version 2.3.1)
- name: RDKit
  role: Chemistry toolkit (2024.09.4) used in the conda environment for validity/connectedness
    filtering and similarity evaluation
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:5b8d4b9a13545f09b3a2f495f707f2daa29827a374362ae7cd02ddfcc279b2d0
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  - SIRIUS
  - torch_geometric
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# mass-spectrum-conditional-molecule-generation

## Summary

Formula-restricted de novo generation of molecular structures from tandem mass spectra using DiffMS, a permutation-invariant discrete graph-diffusion encoder-decoder that generates molecules as graphs (atoms, charges, and a 5-way bond-type adjacency matrix) conditioned on the spectrum and a chemical formula.

## When to use

Use when you have a tandem mass spectrum plus an inferred molecular formula and need candidate molecular structures ranked by top-k accuracy, MCES distance, or Tanimoto similarity against a ground-truth molecule.

## When NOT to use

- You only need candidate molecular formulas rather than full structures — formula-restricted tools like SIRIUS, BUDDY, or MIST-CF address that upstream step
- The target structures exist in standard virtual chemical libraries (PubChem, Human Metabolome Database) where library lookup suffices
- You lack a reliably inferred molecular formula, since DiffMS is formula-restricted (heavy-atom-constrained) and takes the formula as given

## Inputs

- Tandem mass spectrum with inferred molecular formula (formulae inferred from high-resolution MS1 data and isotopic traces, e.g. via SIRIUS)
- Spectra-molecule pair finetuning dataset (NPLIB1/CANOPUS subset of GNPS, or MassSpecGym)
- Fingerprint-molecule pretraining pairs (2.8M from DSSTox, HMDB, COCONUT, MOSES)
- Pretrained MIST encoder checkpoints (Zenodo record 23303289) and pretrained DiffMS end-to-end models/decoders (Zenodo record 23542112)

## Outputs

- Generated molecular structures (graphs of atoms and bonds) per input spectrum
- Top-k candidate molecules per spectrum ranked by sampling frequency
- Evaluation metrics: Top-1/Top-10 accuracy, Top-1/Top-10 minimum MCES distance, Top-1/Top-10 maximum Tanimoto similarity

## How to apply

First process the spectra and molecules with the DiffMS data_processing scripts (00_download_fp2mol_data.sh, 01_download_canopus_data.sh, 02_download_msg_data.sh, 03_preprocess_fp2mol.sh), then pretrain the MIST spectrum encoder on spectrum-to-fingerprint prediction and the diffusion decoder on 2.8M fingerprint-molecule pairs (DSSTox, HMDB, COCONUT, MOSES) via fp2mol_main.py with dataset='fp2mol'. Fine-tune end-to-end on spectra-molecule pairs via spec2mol_main.py (dataset='canopus' for NPLIB1 or 'msg' for MassSpecGym), loading encoder weights by setting general.load_weights in general_default.yaml. The chemical formula restricts the number of heavy atoms and their types, while discrete diffusion on the adjacency matrix with a 5-bond-type vocabulary (no bond, single, double, triple, aromatic) handles bond placement; pretraining on fingerprint-structure pairs bridges the latent spectral embedding and molecular structures. At inference, sample 100 molecules per spectrum, remove invalid or disconnected molecules with RDKit, and rank the top-k candidates by frequency.

## Related tools

- **DiffMS** (Formula-restricted encoder-decoder diffusion model that generates molecular graphs conditioned on mass spectra) — github.com/coleygroup/DiffMS
- **MIST** (Formula transformer used as the spectrum encoder, pretrained to predict 4,096-bit Morgan fingerprints (radius 2, no chirality) from spectra) — coleygroup/MIST
- **SIRIUS** (Infers molecular formulas from high-resolution MS1 data and isotopic traces to condition the formula-restricted generation)
- **PyTorch** (Deep-learning framework for training and running the diffusion model (tested with PyTorch 2.3.1, cuda 11.8))
- **torch_geometric** (Graph library backend for the molecular-graph diffusion model (tested with version 2.3.1))
- **RDKit** (Chemistry toolkit (2024.09.4) used in the conda environment for validity/connectedness filtering and similarity evaluation)

## Examples

```
python src/spec2mol_main.py  # with configs/config.yaml dataset set to 'canopus' (NPLIB1) or 'msg' (MassSpecGym) for spectra-molecule finetuning
```

## Evaluation signals

- Top-1 accuracy of generated molecules against ground truth on NPLIB1 or MassSpecGym (baseline DiffMS reaches state-of-the-art; pretraining roughly doubles NPLIB1 top-1 from 4.36% to 8.34%)
- Minimum MCES distance at Top-1/Top-10 decreases relative to baselines
- Maximum Tanimoto similarity at Top-1/Top-10 increases relative to baselines
- Sampled molecules are valid and connected after RDKit filtering; invalid/disconnected structures are removed before ranking
- Downstream metrics scale consistently with increasing fingerprint-structure pretraining dataset size

## Limitations

- De novo generation of the exact molecular structure remains challenging; structurally close matches, not guaranteed exact structures, are the realistic outcome
- The formula is taken as given from upstream tools, so errors in formula inference (e.g. by SIRIUS or MIST-CF, the latter not considering isotope distributions) propagate into generation
- DiGress, from which the codebase is adapted, has a reported bug where dim_ffy is ignored in GraphTransformer construction (default 2048 used instead of the paper value), with unknown performance impact
- DiffMS codebase expects specific tested versions (PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1, RDKit 2024.09.4, python 3.9); graph_tool and torch_geometric conflict on MacOS in the DiGress lineage

## Evidence

- [methods] Sampling and ranking procedure for generated candidates: [evidence span withheld — anchor status: not_found]
