---
name: elbo-endpoint-prediction-optimization
description: Use when training a discrete diffusion / Markov bridge generative model
  whose denoiser must predict the final clean molecular graph at each diffusion step,
  e.g. MADGEN stage 2, with classifier-free spectrum dropout.
when_to_use_negative:
- The task is scaffold retrieval / ranking of candidate scaffolds — that is MADGEN
  stage 1 contrastive learning, not bridge-model ELBO optimization
- Input molecules are not represented as fully connected graphs with fixed scaffold
  edges, so the absorbing-state Markov bridge formulation does not apply
- Only generated molecules are needed for evaluation and a pre-trained checkpoint
  is already available (Zenodo record 17115068) — training would be redundant
edam_operation: http://edamontology.org/operation_3762
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3474
tools:
- name: MADGEN
  role: Framework whose stage-2 generator is trained with endpoint-prediction ELBO
    over an absorbing-state Markov bridge
  repo: https://github.com/HassounLab/MADGEN
- name: PyTorch
  role: Implementation and checkpoint saving of the graph transformer denoiser
- name: AdamW
  role: Optimizer (learning rate 1e-5) for the ELBO objective
- name: RDKit
  role: Derive Murcko scaffolds to form (scaffold, molecule) training pairs
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:95f7be629c4e2ef987a8032fb73bfaec4125a709a4cd9def9c419fe3cbf7b6fd
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
  edam_operation: http://edamontology.org/operation_3762
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# elbo-endpoint-prediction-optimization

## Summary

Optimizes a scaffold-conditioned Markov bridge molecule generator with the endpoint-prediction ELBO loss, so that the FC-GNN denoiser learns to reconstruct fully connected molecular graphs from fixed Murcko scaffolds under MS/MS spectra guidance.

## When to use

Use when training a discrete diffusion / Markov bridge generative model whose denoiser must predict the final clean molecular graph at each diffusion step, e.g. MADGEN stage 2, with classifier-free spectrum dropout.

## When NOT to use

- The task is scaffold retrieval / ranking of candidate scaffolds — that is MADGEN stage 1 contrastive learning, not bridge-model ELBO optimization
- Input molecules are not represented as fully connected graphs with fixed scaffold edges, so the absorbing-state Markov bridge formulation does not apply
- Only generated molecules are needed for evaluation and a pre-trained checkpoint is already available (Zenodo record 17115068) — training would be redundant

## Inputs

- Processed dataset pickle (e.g. msgym.pkl from Zenodo record 15036069) containing canonical SMILES, normalized MS/MS spectra, and predefined train/val/test splits
- Murcko scaffold strings derived from RDKit for each target molecule
- Peak lists of MS/MS spectra (m/z and intensity values)

## Outputs

- Trained PyTorch checkpoint of the spectra-conditioned Markov bridge generator
- Endpoint-prediction ELBO loss curve showing convergence without divergence

## How to apply

Represent each molecule as a fully connected graph whose edge entries are D-dimensional one-hot vectors (class 0 = non-edge), keeping the retrieved Murcko scaffold edges fixed and generating only edges connecting scaffold atoms to free atoms. Train an absorbing-state Markov bridge model with the endpoint-prediction ELBO loss using a graph transformer of 5 layers over 50 diffusion steps, optimizing with AdamW at learning rate 1e-5. Encode spectrum peaks with MLPs over m/z and intensity, refine them with a self-attention module into peak tokens, and inject those tokens into every message-passing layer via cross-attention. Randomly drop the spectrum condition with probability 0.1 during training so classifier-free guidance can later combine logits as (1+lambda_t)*conditioned - lambda_t*unconditioned. Save the converged model as a PyTorch checkpoint; convergence of the ELBO objective without divergence is the acceptance criterion, as no numeric loss threshold is reported.

## Related tools

- **MADGEN** (Framework whose stage-2 generator is trained with endpoint-prediction ELBO over an absorbing-state Markov bridge) — https://github.com/HassounLab/MADGEN
- **PyTorch** (Implementation and checkpoint saving of the graph transformer denoiser)
- **AdamW** (Optimizer (learning rate 1e-5) for the ELBO objective)
- **RDKit** (Derive Murcko scaffolds to form (scaffold, molecule) training pairs)

## Examples

```
python train.py --config configs/msgym.yaml --model Madgen
```

## Evaluation signals

- Checkpoint file exists and loads in PyTorch without error
- Training ELBO objective converged without divergence (qualitative acceptance; no numeric threshold reported)
- Absorbing-state Markov bridge samples reproduce valid molecular graphs with fixed scaffold edges
- Classifier-free guidance logits combine as (1+lambda_t)*conditioned - lambda_t*unconditioned, indicating the 0.1 spectrum-dropout training was effective

## Limitations

- No numeric training-loss threshold is reported; acceptance is qualitative (ELBO converged without divergence)
- NIST dataset requires separate commercial licensing and is not distributed via Zenodo
- Performance depends on the quality of the retrieved scaffold; the oracle retriever gives stronger results than the predictive one

## Evidence

- [methods] Train the absorbing-state Markov bridge model with endpoint-prediction ELBO loss using a graph transformer with 5 layers and 50 diffusion steps, AdamW optimizer at learning rate 1e-5: [evidence span withheld — anchor status: not_found]
- [methods] We employed the AdamW optimizer with a learning rate of 1×10−5.: [evidence span withheld — anchor status: not_found]
