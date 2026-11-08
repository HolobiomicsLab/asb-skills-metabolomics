---
name: discrete-graph-diffusion-molecule-sampling
description: Use when a tandem mass spectrum plus an inferred chemical formula must
  be converted into de novo candidate structures, or when reproducing DiffMS-style
  top-k accuracy, Tanimoto, and MCES metrics on NPLIB1 or MassSpecGym.
when_to_use_negative:
- Only a molecular formula or fingerprint must be predicted (use the MIST formula
  transformer / spectrum-to-fingerprint pretraining stage instead of the diffusion
  decoder).
- The task is the forward MS problem (spectra from structures) rather than the 'inverse'
  de novo generation problem.
- No chemical formula/heavy-atom composition is available to constrain generation,
  since the diffusion process is formula-restricted by design.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DiffMS
  role: End-to-end formula-restricted encoder-decoder diffusion model that generates
    molecules from spectra; checkpoints loaded via general.load_weights in configs/general/general_default.yaml
  repo: github.com/coleygroup/DiffMS
- name: DiGress
  role: Base discrete-denoising-diffusion graph-generation codebase from which the
    DiffMS decoder and its discrete diffusion process on the adjacency matrix are
    adapted
  repo: cvignac/DiGress
- name: MIST
  role: MIST formula transformer used as the spectrum encoder and for formula inference;
    decoder pretraining can be run separately with fp2mol_main.py (dataset 'fp2mol'
    in config.yaml)
  repo: coleygroup/MIST
- name: RDKit
  role: Validity and connectivity filtering of sampled molecules and Morgan-fingerprint
    Tanimoto similarity computation
- name: torch_geometric 2.3.1
  role: Graph neural network framework supporting the molecular graph diffusion pipeline
    (tested with PyTorch 2.3.1, cuda 11.8)
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_content_hash: sha256:0f41a8aa87647dab977294eeb06c39d5b3580f96ce84849ae6db78317b490643
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
  - torch_geometric 2.3.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 26
---

# Discrete graph diffusion sampling for formula-restricted molecule generation

## Summary

Samples candidate molecular structures from a tandem mass spectrum using a permutation-invariant discrete diffusion model over the molecular adjacency matrix, conditioned on the spectrum and the formula-derived heavy-atom composition (the DiffMS approach built on DiGress).

## When to use

Use when a tandem mass spectrum plus an inferred chemical formula must be converted into de novo candidate structures, or when reproducing DiffMS-style top-k accuracy, Tanimoto, and MCES metrics on NPLIB1 or MassSpecGym.

## When NOT to use

- Only a molecular formula or fingerprint must be predicted (use the MIST formula transformer / spectrum-to-fingerprint pretraining stage instead of the diffusion decoder).
- The task is the forward MS problem (spectra from structures) rather than the 'inverse' de novo generation problem.
- No chemical formula/heavy-atom composition is available to constrain generation, since the diffusion process is formula-restricted by design.

## Inputs

- Tandem mass spectra from the NPLIB1 or MassSpecGym test sets (processed via the data_processing/ scripts 00-03)
- Chemical formula for each spectrum (from MIST or formula-inference tools like SIRIUS/MIST-CF) defining the heavy-atom composition constraint
- Pretrained end-to-end DiffMS or diffusion-decoder checkpoints (Zenodo 23542112); optional MIST encoder checkpoints (Zenodo 23303289)
- Ground-truth molecular structures for evaluation

## Outputs

- 100 sampled molecular structures per spectrum (after invalid/disconnected filtering)
- Ranked top-1 and top-10 candidate lists per spectrum based on sampling frequency
- Top-k accuracy, top-k maximum Tanimoto similarity (RDKit Morgan fingerprints), and top-k minimum MCES distance metrics

## How to apply

Run the formula-restricted decoder (adapted from DiGress) by first conditioning on the spectrum embedding from the MIST spectrum encoder and the atom types derived from the chemical formula, so heavy-atom composition is natively enforced by the discrete diffusion over the molecular adjacency matrix A. Load a pretrained end-to-end DiffMS checkpoint (zenodo.org/records/23542112) or separately load the MIST encoder (zenodo.org/records/23303289) and pretrained decoder via general.load_weights in configs/general/general_default.yaml, then finetune end-to-end with spec2mol_main.py (dataset 'canopus' for NPLIB1 or 'msg' for MassSpecGym). For each test spectrum, sample 100 molecules from the diffusion decoder, use RDKit (2024.09.4) to remove invalid or disconnected molecules, and identify the top-k (k=1, 10) candidates by sampling frequency. Score ranked candidates against ground truth with top-k accuracy, top-k maximum Tanimoto similarity (Morgan fingerprints), and top-k minimum MCES distance following MassSpecGym de novo generation metric definitions. Repetition counts and stochastic-sampling tolerance must be accounted for when comparing against reported Table 1 values.

## Related tools

- **DiffMS** (End-to-end formula-restricted encoder-decoder diffusion model that generates molecules from spectra; checkpoints loaded via general.load_weights in configs/general/general_default.yaml) — github.com/coleygroup/DiffMS
- **DiGress** (Base discrete-denoising-diffusion graph-generation codebase from which the DiffMS decoder and its discrete diffusion process on the adjacency matrix are adapted) — cvignac/DiGress
- **MIST** (MIST formula transformer used as the spectrum encoder and for formula inference; decoder pretraining can be run separately with fp2mol_main.py (dataset 'fp2mol' in config.yaml)) — coleygroup/MIST
- **RDKit** (Validity and connectivity filtering of sampled molecules and Morgan-fingerprint Tanimoto similarity computation)
- **torch_geometric 2.3.1** (Graph neural network framework supporting the molecular graph diffusion pipeline (tested with PyTorch 2.3.1, cuda 11.8))

## Examples

```
python3 src/spec2mol_main.py
```

## Evaluation signals

- Reproduced NPLIB1 top-1 accuracy should match the reported 8.34% (Tanimoto 0.35, MCES 11.95) and top-10 ~15.44% within stochastic-sampling tolerance; MassSpecGym top-1 accuracy 2.30%, Tanimoto 0.28.
- Every emitted candidate passes RDKit validity and connectivity checks (invalid or disconnected molecules removed before top-k ranking).
- Generated molecules satisfy the formula-derived heavy-atom composition constraint imposed by the discrete diffusion.
- Models pretrained on the 2.8M-pair 'combined' fp2mol dataset should outperform zero-pretraining (4.36% top-1) and show monotone scaling with pretraining set size (0, 0.2M, 0.8M, 1.2M, 2.8M).
- File existence and metric-value checks against the reference Table 1 rows in reproduction_results.csv.

## Limitations

- De novo generation of the exact molecular structure remains challenging; the model targets structurally close matches that can still offer insights for domain experts.
- Evaluation is sampling-based (100 molecules per spectrum, top-k by frequency), so reported metrics are subject to stochastic-sampling variance rather than deterministic reproduction.
- The underlying DiGress codebase has a reported bug (dim_ffy ignored in GraphTransformer construction, default 2048 used), and the impact on performance is unknown per the upstream README.

## Evidence

- [methods] Formula-constrained sampling and top-k filtering procedure: [evidence span withheld — anchor status: not_found]
- [readme] Decoder weight loading configuration: [evidence span withheld — anchor status: not_found]
