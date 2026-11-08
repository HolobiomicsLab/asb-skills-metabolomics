---
name: fingerprint-conditioned-molecule-generation
description: Use when generating molecular structures conditioned on an intermediate
  chemical representation (fingerprint) with a diffusion model, or when scaling decoder
  pretraining with large fingerprint-structure pools before spectra-conditioned finetuning.
when_to_use_negative:
- The task is fingerprint prediction from spectra, not structure generation — use
  the MIST spectrum-to-fingerprint encoder training path instead.
- Candidate structures must come from a fixed virtual library with exact matching
  (library search) rather than de novo generation, since generated structures may
  only be structurally close matches.
- Only unlabeled spectra with no reliable chemical formula are available, since the
  pipeline is formula-restricted (heavy-atom-constrained) and takes the formula as
  given from tools like SIRIUS or MIST-CF.
tools:
- name: DiffMS
  role: Diffusion-based encoder-decoder framework whose decoder is pretrained on fingerprint-molecule
    pairs and finetuned end-to-end on spectra-molecule generation
  repo: https://github.com/coleygroup/DiffMS
- name: MIST
  role: Spectrum encoder / formula transformer producing 4,096-bit Morgan fingerprints
    used as the conditioning representation for decoder pretraining
  repo: https://github.com/coleygroup/MIST
- name: DiGress
  role: Discrete denoising diffusion graph-generation codebase from which DiffMS is
    adapted
  repo: https://github.com/cvignac/DiGress
- name: torch_geometric
  role: Graph neural-network/diffusion library used to implement the molecular graph
    diffusion (tested with 2.3.1)
- name: RDKit
  role: Molecule validity and connectivity checks during sampling, and fingerprint
    handling (tested with rdkit=2024.09.4)
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:924998f2b8d7e7723b579311d744b28772de0c3df8473d381eb780e5f6bae658
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - COCONUT
  - DSSTox
  - DiGress
  - DiffMS
  - HMDB
  - MIST
  - MOSES
  - NPLIB1
  - PyTorch
  - RDKit
  - torch_geometric
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# Fingerprint-conditioned molecule generation

## Summary

Pretrain a discrete graph-diffusion decoder to map Morgan molecular fingerprints to molecular structures, then finetune end-to-end from mass spectra. This exploits the near-unlimited supply of fingerprint-structure pairs to pretrain the generative decoder before spectra-molecule finetuning.

## When to use

Use when generating molecular structures conditioned on an intermediate chemical representation (fingerprint) with a diffusion model, or when scaling decoder pretraining with large fingerprint-structure pools before spectra-conditioned finetuning.

## When NOT to use

- The task is fingerprint prediction from spectra, not structure generation — use the MIST spectrum-to-fingerprint encoder training path instead.
- Candidate structures must come from a fixed virtual library with exact matching (library search) rather than de novo generation, since generated structures may only be structurally close matches.
- Only unlabeled spectra with no reliable chemical formula are available, since the pipeline is formula-restricted (heavy-atom-constrained) and takes the formula as given from tools like SIRIUS or MIST-CF.

## Inputs

- Fingerprint-to-molecule pretraining pairs (up to 2.8M pairs from DSSTox, HMDB, COCONUT, MOSES) prepared via data_processing/00_download_fp2mol_data.sh and 03_preprocess_fp2mol.sh
- 4,096-bit Morgan fingerprints (radius 2, without chirality) as decoder conditioning
- Spectra-to-molecule finetuning data: NPLIB1 (CANOPUS subset of GNPS) or MassSpecGym
- MIST encoder checkpoint (Zenodo record 23303289)

## Outputs

- Pretrained diffusion-decoder checkpoints for each pretraining-pool size (Zenodo record 23542112 for released versions)
- Finetuned end-to-end spectra-to-structure generative model
- Per-spectrum top-k molecule predictions (100 samples per spectrum, invalid/disconnected removed, ranked by frequency)
- Scaling table of top-1/top-10 accuracy across pretraining sizes (2.8M-pair variant: ~8.34% top-1, ~15.44% top-10 on NPLIB1)

## How to apply

Build the 'combined' pretraining dataset from DSSTox, HMDB, COCONUT, and MOSES fingerprint-molecule pairs, removing all NPLIB1 test and validation molecules. Subsample the pool (e.g., 0, 0.2M, 0.8M, 1.2M, 2.8M pairs) and pretrain the permutation-invariant diffusion decoder with fp2mol_main.py, setting dataset='fp2mol' in configs/config.yaml; the decoder conditions generation on 4,096-bit Morgan fingerprints (radius 2, no chirality) as produced by the MIST formula transformer. Pretraining uses discrete diffusion on the molecular adjacency matrix (DiGress-style), which natively handles the predefined heavy-atom composition fixed by the candidate formula. Then initialize the end-to-end model with the MIST spectrum-encoder weights plus the pretrained decoder and finetune on spectra-to-molecule generation with spec2mol_main.py (dataset 'canopus' for NPLIB1 or 'msg' for MassSpecGym). At inference, sample 100 molecules per spectrum, drop invalid or disconnected molecules, and rank top-k molecules by frequency.

## Related tools

- **DiffMS** (Diffusion-based encoder-decoder framework whose decoder is pretrained on fingerprint-molecule pairs and finetuned end-to-end on spectra-molecule generation) — https://github.com/coleygroup/DiffMS
- **MIST** (Spectrum encoder / formula transformer producing 4,096-bit Morgan fingerprints used as the conditioning representation for decoder pretraining) — https://github.com/coleygroup/MIST
- **DiGress** (Discrete denoising diffusion graph-generation codebase from which DiffMS is adapted) — https://github.com/cvignac/DiGress
- **torch_geometric** (Graph neural-network/diffusion library used to implement the molecular graph diffusion (tested with 2.3.1))
- **RDKit** (Molecule validity and connectivity checks during sampling, and fingerprint handling (tested with rdkit=2024.09.4))

## Examples

```
python src/fp2mol_main.py  # with dataset set to 'fp2mol' in configs/config.yaml, then finetune: python src/spec2mol_main.py (dataset 'canopus')
```

## Evaluation signals

- The 2.8M-pair pretrained + finetuned model should reach approximately 8.34% top-1 and 15.44% top-10 accuracy on NPLIB1.
- Scaling check: downstream NPLIB1 metrics (top-1/top-10 accuracy, minimum MCES distance, maximum Tanimoto similarity) should improve monotonically (consistent scaling) as pretraining pool size grows from 0 to 2.8M pairs.
- Every sampled molecule must be chemically valid and connected after filtering; invalid or disconnected structures are discarded before top-k frequency ranking.
- Generated structures must respect the formula-constrained (heavy-atom) composition implied by the input formula.
- Ablation sanity check: the 0-pair (no pretraining) variant should be clearly worse than pretrained variants, confirming pretraining gains.

## Limitations

- De novo generation of the exact molecular structure remains challenging; outputs are structurally close matches useful as leads, not guaranteed identifications.
- Pretraining performance depends on removing all NPLIB1 test and validation molecules from the 'combined' fingerprint-molecule pool to avoid leakage.
- Evaluation requires sampling 100 molecules per spectrum, which makes inference comparatively expensive; the MIST-CF conditioning alternative does not consider isotope distributions.
- DiffMS is adapted from DiGress, whose README notes known code bugs (e.g., dim_ffy ignored in GraphTransformer construction) that may affect reported configurations.

## Evidence

- [methods] we sample 100 molecules for each spectrum, remove invalid or disconnected molecules, and identify the top-k molecules based on frequency: [evidence span withheld — anchor status: not_found]
- [readme] It outputs 4,096-bit Morgan fingerprints (radius 2, without chirality).: [evidence span withheld — anchor status: external_doc]
