---
name: checkpoint-based-model-inference
description: Use when a task requires reproducing reported de novo generation metrics
  (NPLIB1 top-1 accuracy 8.34%, MassSpecGym 2.30%) from pretrained DiffMS weights
  rather than training from scratch, given processed NPLIB1/MassSpecGym test spectra.
when_to_use_negative:
- You intend to train or finetune the model from scratch (e.g., fingerprint-molecule
  pretraining via fp2mol_main.py) rather than run inference with released weights.
- Input spectra lack reliable chemical formulas, since DiffMS generation is formula-restricted/heavy-atom-constrained
  and conditions on formula-derived atom types.
- The dataset is not one of the supported benchmark configurations ('msg', 'canopus')
  handled by the released configs.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DiffMS
  role: Pretrained encoder-decoder diffusion model whose checkpoints are loaded and
    run for spectra-to-molecule inference
  repo: https://github.com/coleygroup/DiffMS
- name: MIST
  role: Formula transformer encoder whose checkpoints can be loaded from Zenodo when
    encoder/decoder are used separately
  repo: https://github.com/coleygroup/MIST
- name: RDKit 2024.09.4
  role: Validity and connectivity checks on sampled molecules and Morgan fingerprint
    computation for Tanimoto scoring
- name: PyTorch 2.3.1
  role: Runtime for executing the diffusion decoder sampling with the loaded checkpoints
- name: torch_geometric 2.3.1
  role: Graph machinery for the molecular graph diffusion decoder at inference time
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:b42c4e70e8ec01c932649fd0db4c069fb7c20a3569cc2316b5e91a615ad9ecbc
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiffMS
  - MIST
  - PyTorch
  - PyTorch 2.3.1
  - RDKit 2024.09.4
  - torch_geometric 2.3.1
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# checkpoint-based-model-inference

## Summary

Run published DiffMS pretrained checkpoints (end-to-end models/decoders from Zenodo 23542112, MIST encoder checkpoints from Zenodo 23303289) to generate candidate molecular structures from tandem mass spectra without retraining, by wiring checkpoint paths into the config and executing the diffusion decoder inference.

## When to use

Use when a task requires reproducing reported de novo generation metrics (NPLIB1 top-1 accuracy 8.34%, MassSpecGym 2.30%) from pretrained DiffMS weights rather than training from scratch, given processed NPLIB1/MassSpecGym test spectra.

## When NOT to use

- You intend to train or finetune the model from scratch (e.g., fingerprint-molecule pretraining via fp2mol_main.py) rather than run inference with released weights.
- Input spectra lack reliable chemical formulas, since DiffMS generation is formula-restricted/heavy-atom-constrained and conditions on formula-derived atom types.
- The dataset is not one of the supported benchmark configurations ('msg', 'canopus') handled by the released configs.

## Inputs

- Pretrained DiffMS end-to-end model/decoder checkpoints (Zenodo record 23542112)
- MIST encoder checkpoints (Zenodo record 23303289), if loading encoder/decoder separately
- Processed NPLIB1 (CANOPUS) and MassSpecGym test spectra with chemical formulas
- configs/general/general_default.yaml with general.load_weights set to the checkpoint path
- configs/config.yaml with dataset set to 'msg' or 'canopus'

## Outputs

- 100 sampled molecules per test spectrum from the diffusion decoder
- Filtered and frequency-ranked top-k (k=1, 10) candidate molecular structures per spectrum
- Top-k accuracy, top-k max Tanimoto similarity (RDKit Morgan fingerprints), and top-k min MCES distance per benchmark
- Reproduction results comparable to Table 1 (e.g., NPLIB1 top-1 accuracy 8.34%, MCES 11.95, Tanimoto 0.35)

## How to apply

Download the pretrained end-to-end DiffMS checkpoints from zenodo.org/records/23542112 (and MIST encoder checkpoints from zenodo.org/records/23303289 if the encoder and decoder are loaded separately). After running the data_processing/ scripts 00-03 to build the 'msg' or 'canopus' datasets, set the checkpoint path via general.load_weights in configs/general/general_default.yaml and set configs/config.yaml dataset to 'msg' for MassSpecGym or 'canopus' for NPLIB1. For each test spectrum, run the graph diffusion decoder to sample 100 molecules conditioned on the spectrum and the formula-derived heavy-atom types. Remove invalid or disconnected molecules with RDKit and rank the top-k (k=1, 10) candidates by frequency. Score ranked lists against ground truth with top-k accuracy, maximum Tanimoto similarity (RDKit Morgan fingerprints), and minimum MCES distance, then compare against the paper's Table 1 values within stochastic-sampling tolerance.

## Related tools

- **DiffMS** (Pretrained encoder-decoder diffusion model whose checkpoints are loaded and run for spectra-to-molecule inference) — https://github.com/coleygroup/DiffMS
- **MIST** (Formula transformer encoder whose checkpoints can be loaded from Zenodo when encoder/decoder are used separately) — https://github.com/coleygroup/MIST
- **RDKit 2024.09.4** (Validity and connectivity checks on sampled molecules and Morgan fingerprint computation for Tanimoto scoring)
- **PyTorch 2.3.1** (Runtime for executing the diffusion decoder sampling with the loaded checkpoints)
- **torch_geometric 2.3.1** (Graph machinery for the molecular graph diffusion decoder at inference time)

## Examples

```
conda create -y -c conda-forge -n diffms rdkit=2024.09.4 python=3.9; conda activate diffms; pip install torch==2.3.1 --index-url https://download.pytorch.org/whl/cu118; pip install -e .; python src/spec2mol_main.py
```

## Evaluation signals

- Reproduced NPLIB1 top-1 accuracy matches the reported 8.34% (MCES 11.95, Tanimoto 0.35) within stochastic-sampling tolerance.
- Reproduced MassSpecGym top-1 accuracy matches the reported 2.30% (Tanimoto 0.28) within stochastic-sampling tolerance.
- All sampled molecules pass RDKit validity and connectivity checks before ranking.
- Checkpoint path resolves correctly via general.load_weights in general_default.yaml (file-existence check).
- Exactly 100 candidate molecules are sampled per spectrum before invalid/disconnected removal.

## Limitations

- Sampling-based evaluation is stochastic; reproduced metrics match Table 1 only within sampling tolerance, not exactly.
- Checkpoint availability is fragile in general (cf. the DiGress README where the author's Drive checkpoints were lost), so Zenodo records should be verified before reliance.
- DiGress-derived code has a known bug: dim_ffy is ignored in GraphTransformer construction and the default 2048 is used, with unknown performance impact.

## Evidence

- [readme] readme checkpoint availability: "Pretrained DiffMS end-to-end models and decoders are available here"
- [readme] readme config wiring: [evidence span withheld — anchor status: not_found]
- [readme] readme dataset switch: "You will also need to set the dataset in [config.yaml](configs/config.yaml) to 'msg' for MassSpecGym or 'canopus' for NPLIB1"
- [methods] methods sampling protocol: [evidence span withheld — anchor status: not_found]
- [methods] methods environment: "This code was tested with PyTorch 2.3.1, cuda 11.8 and torch_geometrics 2.3.1"
