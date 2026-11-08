---
name: scaffold-conditioned-markov-bridge-training
description: Use when building a spectra-conditioned de novo molecule generator whose
  output must contain a fixed Murcko scaffold subgraph, e.g. training on NIST23, CANOPUS,
  or MassSpecGym-style pickles of canonical SMILES and normalized MS/MS spectra.
when_to_use_negative:
- When the task is retrieving or ranking scaffolds from spectra rather than generating
  molecules — that is the separate scaffold_retrieval contrastive-learning stage (cand_rank_msg.py,
  cand_rank_canopus.py).
- When the target molecule is already known and only spectrum-to-structure lookup
  is needed; MADGEN's oracle retriever plus generation is for de novo generation,
  not annotation lookup.
- When no scaffold constraint is desired (unconstrained de novo generation); the whole
  rationale is that constraining the generation search space via the scaffold reduces
  complexity.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3418
- http://edamontology.org/topic_0091
tools:
- name: MADGEN
  role: Provides the scaffold-conditioned Markov bridge model, FC-GNN with cross-attention
    peak tokens, and the endpoint-prediction ELBO training objective being trained
    here.
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts Murcko scaffolds from molecular graphs to build the (scaffold, molecule)
    training pairs.
- name: PyTorch
  role: Implementation framework; training is done with the AdamW optimizer and the
    checkpoint is a PyTorch file.
- name: AdamW
  role: Optimizer used for the endpoint-prediction ELBO training at learning rate
    1e-5.
- name: conda
  role: Environment setup (python=3.9, rdkit=2023.09.5) before training.
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:8b880808ff92add7a2578bfed14c010c75881183a949a3dc31b8ccf79fcf4e23
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - AdamW
  - MADGEN
  - PyTorch
  - RDKit
  - conda
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3418
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scaffold-conditioned-markov-bridge-training

## Summary

Trains MADGEN's second-stage generator, an absorbing-state Markov bridge model that turns a Murcko scaffold graph plus an MS/MS spectrum into the target molecule, using endpoint-prediction ELBO loss and classifier-free guidance from the spectrum.

## When to use

Use when building a spectra-conditioned de novo molecule generator whose output must contain a fixed Murcko scaffold subgraph, e.g. training on NIST23, CANOPUS, or MassSpecGym-style pickles of canonical SMILES and normalized MS/MS spectra.

## When NOT to use

- When the task is retrieving or ranking scaffolds from spectra rather than generating molecules — that is the separate scaffold_retrieval contrastive-learning stage (cand_rank_msg.py, cand_rank_canopus.py).
- When the target molecule is already known and only spectrum-to-structure lookup is needed; MADGEN's oracle retriever plus generation is for de novo generation, not annotation lookup.
- When no scaffold constraint is desired (unconstrained de novo generation); the whole rationale is that constraining the generation search space via the scaffold reduces complexity.

## Inputs

- Processed dataset pickle (e.g. msgym.pkl, canopus.pkl) with canonical SMILES, normalized MS/MS spectra, and predefined train/val/test splits
- Murcko scaffold-molecule pairs derived from the SMILES via RDKit
- Normalized MS/MS spectra with low-intensity peaks removed

## Outputs

- PyTorch checkpoint of the trained scaffold-conditioned Markov bridge generator (e.g. checkpoints/msgym_best.pt)
- Trained FC-GNN / graph-transformer weights usable by sample.py for test-time generation

## How to apply

Load a processed dataset pickle (e.g. msgym.pkl from Zenodo record 15036069) holding canonical SMILES, normalized MS/MS spectra, and predefined train/val/test splits, then derive Murcko scaffolds with RDKit to form (scaffold, molecule) pairs. Represent each molecule as a fully connected graph whose edges are D-dimensional one-hot vectors (class 0 = non-edge), keep scaffold edges fixed, and generate only edges linking scaffold atoms to free atoms. Tokenize the spectrum by encoding m/z and intensity peaks with MLPs refined through self-attention into peak tokens, and inject these tokens into every message-passing layer of the FC-GNN via cross-attention (node-only, edge-only, or combined variants). Train the absorbing-state bridge with a graph transformer of 5 layers and 50 diffusion steps using the endpoint-prediction ELBO loss with AdamW at learning rate 1e-5, dropping the spectrum condition with probability 0.1 during training so classifier-free guidance can combine conditioned and unconditioned logits as (1+lambda_t)*conditioned - lambda_t*unconditioned. Save the resulting weights as a PyTorch checkpoint for downstream sampling.

## Related tools

- **MADGEN** (Provides the scaffold-conditioned Markov bridge model, FC-GNN with cross-attention peak tokens, and the endpoint-prediction ELBO training objective being trained here.) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts Murcko scaffolds from molecular graphs to build the (scaffold, molecule) training pairs.)
- **PyTorch** (Implementation framework; training is done with the AdamW optimizer and the checkpoint is a PyTorch file.)
- **AdamW** (Optimizer used for the endpoint-prediction ELBO training at learning rate 1e-5.)
- **conda** (Environment setup (python=3.9, rdkit=2023.09.5) before training.)

## Examples

```
python train.py --config configs/msgym.yaml --model Madgen
```

## Evaluation signals

- The saved checkpoint file exists and loads in PyTorch (e.g. checkpoints/msgym_best.pt).
- Training ELBO converged without divergence (qualitative acceptance; the paper reports no numeric loss threshold).
- Sampled molecules from the checkpoint decode to valid SMILES and respect the fixed scaffold edges when generation starts from the scaffold.
- Downstream evaluation_generation.py reports Top-K InChI-key accuracy and maximum Tanimoto similarity (Morgan radius=2, 2048 bits) consistent with published MADGEN results.
- Classifier-free guidance behaves as expected: conditioned logits weighted by (1+lambda_t) dominate the unconditioned logits scaled by lambda_t.

## Limitations

- Scaffold retrieval errors propagate: with a predictive retriever the generator sees the wrong scaffold, so oracle-retriever results are an upper bound.
- The generator can only place atoms/edges connecting the fixed scaffold to free atoms; structures whose scaffold differs from the retrieved Murcko scaffold cannot be produced.
- MSNovelist-style baselines report accuracy metrics only; other similarity/MCES measures are unavailable, limiting cross-method comparison.
- NIST23 data is commercially licensed and must be acquired separately; only msgym.pkl and canopus.pkl are freely downloadable from Zenodo.

## Evidence

- [methods] We employed the AdamW optimizer with a learning rate of 1×10−5.: [evidence span withheld — anchor status: not_found]
