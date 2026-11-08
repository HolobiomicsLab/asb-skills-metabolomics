---
name: classifier-free-guidance-training
description: Use when training a graph-based generative model for de novo molecular
  structure generation where an MS/MS spectrum must guide generation from a fixed
  scaffold, and guidance strength at sampling must be tunable without retraining a
  separate classifier.
when_to_use_negative:
- The condition is a discrete label or embedding rather than a full MS/MS spectrum,
  and a simpler conditional generative model without guidance-mixing suffices
- You only need scaffold retrieval/ranking rather than de novo generation from the
  scaffold — that is handled by the contrastive retrieval module, not this training
  stage
- The NIST dataset has not been licensed and no processed pickle is available for
  the target dataset
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_3474
tools:
- name: MADGEN
  role: The scaffold-conditioned Markov bridge model being trained with spectrum-condition
    dropout
  repo: https://github.com/HassounLab/MADGEN
- name: PyTorch
  role: Framework implementing the graph transformer and AdamW training loop
- name: RDKit
  role: Derives Murcko scaffolds to form (scaffold, molecule) training pairs
- name: AdamW
  role: Optimizer used at learning rate 1e-5 for the ELBO objective
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:8ca16e133150f38d9e441c7baf62016ac8d4b25215e142f45ad0422f770679cf
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
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# classifier-free-guidance-training

## Summary

Trains the MADGEN scaffold-conditioned Markov bridge generator so that the model works both with and without an MS/MS spectrum condition, enabling classifier-free guidance at sampling time by mixing conditional and unconditional logits.

## When to use

Use when training a graph-based generative model for de novo molecular structure generation where an MS/MS spectrum must guide generation from a fixed scaffold, and guidance strength at sampling must be tunable without retraining a separate classifier.

## When NOT to use

- The condition is a discrete label or embedding rather than a full MS/MS spectrum, and a simpler conditional generative model without guidance-mixing suffices
- You only need scaffold retrieval/ranking rather than de novo generation from the scaffold — that is handled by the contrastive retrieval module, not this training stage
- The NIST dataset has not been licensed and no processed pickle is available for the target dataset

## Inputs

- Processed dataset pickle (e.g. msgym.pkl from Zenodo record 15036069) containing canonical SMILES, normalized MS/MS spectra, and predefined train/val/test splits
- Murcko scaffold-molecule training pairs derived from canonical SMILES with RDKit
- Molecules represented as fully connected graphs with D-dimensional one-hot edge vectors (class 0 = non-edge)

## Outputs

- PyTorch checkpoint of the trained MADGEN generator (e.g. checkpoints/msgym_best.pt) usable for conditional or unconditional sampling
- Training ELBO loss trajectory indicating convergence

## How to apply

Train an absorbing-state Markov bridge model whose graph transformer (5 layers, 50 diffusion steps) predicts edge endpoints on a fully connected graph where scaffold edges are fixed and only scaffold-to-free-atom edges are generated. During each training step, randomly drop the MS/MS spectrum condition with probability 0.1 so the network learns both conditional and unconditional endpoint predictions from the same weights. Spectral guidance is injected via cross-attention: peaks are encoded with MLPs over m/z and intensity, refined into peak tokens by a self-attention module, and mapped into every message-passing layer of the FC-GNN. Optimize the endpoint-prediction ELBO loss with AdamW at learning rate 1e-5. At sampling, combine conditional and unconditional logits as (1 + lambda_t) * conditioned - lambda_t * unconditioned, letting lambda_t be tuned post-training without any auxiliary model.

## Related tools

- **MADGEN** (The scaffold-conditioned Markov bridge model being trained with spectrum-condition dropout) — https://github.com/HassounLab/MADGEN
- **PyTorch** (Framework implementing the graph transformer and AdamW training loop)
- **RDKit** (Derives Murcko scaffolds to form (scaffold, molecule) training pairs)
- **AdamW** (Optimizer used at learning rate 1e-5 for the ELBO objective)

## Examples

```
python train.py --config configs/msgym.yaml --model Madgen
```

## Evaluation signals

- Checkpoint file exists and loads successfully in PyTorch
- Training ELBO loss converged without divergence (qualitative acceptance; no numeric threshold reported)
- At sampling time, varying lambda_t in the logits combination (1 + lambda_t)*conditioned - lambda_t*unconditioned changes generation behavior, confirming both branches are learned
- Sampling from the checkpoint (e.g. n_samples 50, n_steps 100) produces valid SMILES with scaffold coverage above chance on the held-out test split

## Limitations

- No numeric training-loss threshold is reported; convergence is judged qualitatively
- Random condition dropout (p=0.1) during training means the unconditional branch is only weakly learned and sampling quality depends on lambda_t tuning
- NIST23 data is commercially licensed and must be obtained separately; only MSGym and CANOPUS pickles plus pre-trained checkpoints are openly released

## Evidence

- [abstract] MADGEN operates in two stages: scaffold retrieval and spectra-conditioned molecular generation starting with the scaffold: "MADGEN operates in two stages: scaffold retrieval and spectra-conditioned molecular generation starting with the scaffold"
- [methods] We employed the AdamW optimizer with a learning rate of 1×10−5: [evidence span withheld — anchor status: not_found]
- [methods] We construct the lookup table by extracting the scaffold from the molecular graph representation using RDKit: "We construct the lookup table by extracting the scaffold from the molecular graph representation using RDKit."
- [readme] README training command and pre-trained checkpoints: "python train.py --config configs/msgym.yaml --model Madgen"
- [readme] README checkpoint availability: "Pre-trained model checkpoints are available at: [MADGEN Checkpoints](https://zenodo.org/records/17115068)"
