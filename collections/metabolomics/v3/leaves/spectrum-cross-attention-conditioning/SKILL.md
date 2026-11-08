---
name: spectrum-cross-attention-conditioning
description: Use when training or adapting a Markov-bridge molecular generator in
  which a fully connected (scaffold + molecule) graph must be conditioned on normalized
  MS/MS peak lists, and the spectrum should influence each layer of generation.
when_to_use_negative:
- The task only needs scaffold retrieval/ranking from the spectrum — use the contrastive
  scaffold-retrieval module, not the cross-attention generation stage.
- The generation model conditions on the spectrum only once (e.g. token decoding as
  in Spec2Mol) rather than layer-wise graph conditioning — this skill's cross-attention
  design is specific to the FC-GNN Markov bridge.
- The input is a precomputed retrieval rank file (ranks_msgym_pred.pkl / ranks_canopus_pred.pkl)
  with no spectra to encode — spectrum tokens cannot be built.
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MADGEN
  role: The two-stage framework whose second stage implements the spectrum-cross-attention-conditioned
    Markov bridge generator.
  repo: https://github.com/HassounLab/MADGEN
- name: PyTorch
  role: Framework used to implement the FC-GNN, peak-token encoders, self-attention
    module, and the absorbing-state Markov bridge training loop.
- name: RDKit 2023.09.5
  role: Derives Murcko scaffolds from molecules to build the (scaffold, molecule)
    graph pairs whose edges the conditioned generator completes.
- name: AdamW
  role: Optimizer for the conditioned generation model at learning rate 1×10−5.
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:46804b8e79d29bd5ddcb58351dfe140a3d47f73dd48746fc766f6bb4e8c89d73
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
  - RDKit 2023.09.5
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectrum-cross-attention-conditioning

## Summary

Injects MS/MS peak tokens into every message-passing layer of the MADGEN fully-connected graph neural network via cross-attention so that the mass spectrum guides scaffold-conditioned de novo molecule generation from start to finish, rather than only initializing the graph.

## When to use

Use when training or adapting a Markov-bridge molecular generator in which a fully connected (scaffold + molecule) graph must be conditioned on normalized MS/MS peak lists, and the spectrum should influence each layer of generation.

## When NOT to use

- The task only needs scaffold retrieval/ranking from the spectrum — use the contrastive scaffold-retrieval module, not the cross-attention generation stage.
- The generation model conditions on the spectrum only once (e.g. token decoding as in Spec2Mol) rather than layer-wise graph conditioning — this skill's cross-attention design is specific to the FC-GNN Markov bridge.
- The input is a precomputed retrieval rank file (ranks_msgym_pred.pkl / ranks_canopus_pred.pkl) with no spectra to encode — spectrum tokens cannot be built.

## Inputs

- Processed dataset pickle (e.g. msgym.pkl from Zenodo record 15036069) with canonical SMILES, normalized MS/MS spectra (m/z, intensity peaks), and predefined train/val/test splits
- Murcko scaffold graphs extracted with RDKit from the training molecules (scaffold edges kept fixed)
- Checkpoint configuration YAML (e.g. configs/msgym.yaml) specifying model_dim, num_layers, batch_size, learning_rate, num_epochs

## Outputs

- PyTorch checkpoint of the spectrum-conditioned Markov bridge generator (e.g. checkpoints/msgym_best.pt on Zenodo record 17115068)
- Generated molecular structures (SMILES/graphs) conditioned on the input spectrum and retrieved scaffold
- Per-generation log with sampling seed, n_samples, n_steps and scaffold SMILES for downstream evaluation

## How to apply

Represent each peak of the normalized MS/MS spectrum (m/z and intensity) with MLP encoders, then refine the peak embeddings with a self-attention module to form peak tokens. Map these peak tokens into every message-passing layer of the fully connected graph neural network (FC-GNN) via cross-attention, using node-only, edge-only, or combined node-and-edge variants depending on what the generation model needs to condition. Train the absorbing-state Markov bridge with endpoint-prediction ELBO loss using a graph transformer of 5 layers over 50 diffusion steps with AdamW at learning rate 1e-5, randomly dropping the spectrum condition with probability 0.1 to enable classifier-free guidance whose logits are combined as (1+lambda_t)*conditioned - lambda_t*unconditioned. The rationale, from MADGEN, is that integrating spectral information throughout the whole generation process constrains the search space and improves generation accuracy versus conditioning only at the start.

## Related tools

- **MADGEN** (The two-stage framework whose second stage implements the spectrum-cross-attention-conditioned Markov bridge generator.) — https://github.com/HassounLab/MADGEN
- **PyTorch** (Framework used to implement the FC-GNN, peak-token encoders, self-attention module, and the absorbing-state Markov bridge training loop.)
- **RDKit 2023.09.5** (Derives Murcko scaffolds from molecules to build the (scaffold, molecule) graph pairs whose edges the conditioned generator completes.)
- **AdamW** (Optimizer for the conditioned generation model at learning rate 1×10−5.)

## Examples

```
python train.py --config configs/msgym.yaml --model Madgen && python sample.py --config configs/msgym.yaml --checkpoint checkpoints/msgym_best.pt --n_samples 100 --n_steps 200 --table_name msgym_generation
```

## Evaluation signals

- Checkpoint file exists and loads in PyTorch without error.
- Training ELBO (endpoint-prediction loss) converges without divergence over the 50 diffusion steps.
- Sampling with a fixed sampling_seed (e.g. 42) and n_steps=100 produces valid SMILES that can be canonicalized by RDKit.
- Top-K accuracy (InChI key match) and maximum Tanimoto similarity (Morgan fingerprints radius=2, 2048 bits) on the test split are comparable to the paper's MADGEN results for the dataset.
- Scaffold coverage: the fraction of generated molecules whose scaffold matches the conditioned (retrieved or oracle) scaffold is high.

## Limitations

- MSNovelist baseline results are limited to accuracy metrics; similarity and MCES measures are not available for comparison.
- The NIST23 dataset is commercially available and requires separate licensing, limiting direct reproduction on that split.
- The spectrum condition is randomly dropped with probability 0.1 during training to enable classifier-free guidance, so unconditional generations can occur if guidance weight is mis-set.

## Evidence

- [methods] Tokenize each spectrum by encoding peaks with MLPs over m/z and intensity, refining them with a self-attention module into peak tokens, and map these tokens into every message-passing layer of the fully connected graph neural network (FC-GNN) via cross-attention: [evidence span withheld — anchor status: not_found]
- [methods] randomly dropping the spectrum condition with probability 0.1 during training to enable classifier-free guidance with logits combined as (1+lambda_t)*conditioned - lambda_t*unconditioned: [evidence span withheld — anchor status: not_found]
- [methods] Train the absorbing-state Markov bridge model with endpoint-prediction ELBO loss using a graph transformer with 5 layers and 50 diffusion steps: [evidence span withheld — anchor status: not_found]
- [methods] We employed the AdamW optimizer with a learning rate of 1×10−5.: [evidence span withheld — anchor status: not_found]
