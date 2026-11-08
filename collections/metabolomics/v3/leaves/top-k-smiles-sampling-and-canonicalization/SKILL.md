---
name: top-k-smiles-sampling-and-canonicalization
description: 'Use when evaluating a substructure-to-structure or spectrum-to-structure
  NMR2Struct transformer: you need stochastic SMILES generation from an input vector
  and a canonical, order-invariant way to count exact-match accuracy.'
when_to_use_negative:
- You need a single deterministic structure per input rather than a ranked/stochastic
  candidate set — the 15-sample protocol is inherently probabilistic and does not
  guarantee the exact molecule.
- The task is substructure inference from spectra (use infer_multitask_model_substructures
  instead of SMILES generation).
- Molecules exceed the model's validated scope of 19 heavy atoms, where substructure-to-structure
  accuracy falls below 80%.
edam_operation: http://edamontology.org/operation_3429
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Provides the nmr_infer entry point, YAML configuration system, token conventions,
    and top-k sampling generation loop for SMILES prediction
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes both generated and target SMILES strings so exact-match scoring
    is order-invariant
- name: transformer (encoder-decoder)
  role: Autoregressively generates SMILES tokens via top-k random sampling (k=5) from
    the substructure/spectrum encoder representation
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:8be938a66a8365c42c452038925c71909fcf0f9f8299ef122f9068d560f9d500
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - transformer (encoder-decoder)
  edam_operation: http://edamontology.org/operation_3429
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Top-k SMILES Sampling and Canonicalization

## Summary

Generates multiple candidate molecular structures from a trained NMR2Struct transformer by sampling SMILES token-by-token with top-k random sampling (k=5), then canonicalizes the 15 predictions per input with RDKit to enable exact-match scoring against the target molecule.

## When to use

Use when evaluating a substructure-to-structure or spectrum-to-structure NMR2Struct transformer: you need stochastic SMILES generation from an input vector and a canonical, order-invariant way to count exact-match accuracy.

## When NOT to use

- You need a single deterministic structure per input rather than a ranked/stochastic candidate set — the 15-sample protocol is inherently probabilistic and does not guarantee the exact molecule.
- The task is substructure inference from spectra (use infer_multitask_model_substructures instead of SMILES generation).
- Molecules exceed the model's validated scope of 19 heavy atoms, where substructure-to-structure accuracy falls below 80%.

## Inputs

- Binary (or integer index list of 1-957) substructure presence vector from a target molecule
- Trained NMR2Struct transformer checkpoint and alphabet (checkpoints/alphabet.npy or example_configs/alphabet.npy)
- YAML configuration file with global_args, model, data, and inference sections
- Canonicalized target SMILES string (RDKit-canonicalized) for exact-match scoring

## Outputs

- 15 sampled SMILES strings per input (hdf5 file of raw predictions written to the save directory)
- RDKit-canonicalized SMILES predictions
- Exact-match accuracy metrics (per-input and aggregate, optionally binned by heavy-atom count) and Tanimoto similarities of best incorrect predictions

## How to apply

Instantiate the encoder-decoder transformer with the token conventions from the README (source_size = number of substructures + 1, e.g., 958; tgt_start_token = alphabet length + 1; tgt_stop_token = alphabet length + 2) and load the trained checkpoint plus the alphabet file (e.g., example_configs/alphabet.npy for the original paper model). At inference, autoregressively decode SMILES token-by-token using top-k random sampling with k=5, generating 15 predictions per input and terminating each sequence when the stop token is predicted. Canonicalize every generated SMILES string with RDKit, then score success by comparing canonicalized predictions against the canonicalized target SMILES for an exact match; optionally compute Tanimoto similarity (Morgan/fingerprint-based) for incorrect predictions. Run inference with the nmr_infer entry point, setting inference.splits and the pred_gen_opts alphabet field correctly in the YAML config.

## Related tools

- **NMR2Struct** (Provides the nmr_infer entry point, YAML configuration system, token conventions, and top-k sampling generation loop for SMILES prediction) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes both generated and target SMILES strings so exact-match scoring is order-invariant)
- **transformer (encoder-decoder)** (Autoregressively generates SMILES tokens via top-k random sampling (k=5) from the substructure/spectrum encoder representation) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- Reproduce the paper's aggregate benchmark: 93.2% exact-match accuracy on the test set (top-k sampling, k=5, 15 predictions per input).
- Reproduce the per-heavy-atom accuracy trend: 96.0% at 10 heavy atoms, 92.8% at 17 heavy atoms, below 80% at 18-19 heavy atoms.
- Every generated SMILES must terminate with the predicted stop token and be parseable/canonicalizable by RDKit; unparseable outputs indicate token-convention errors (tgt_start_token = alphabet length + 1, tgt_stop_token = alphabet length + 2).
- For incorrect predictions, expect ~89.6% of best incorrect predictions to have Tanimoto similarity >= 0.50 (average 0.68); much lower similarity suggests a mis-trained or mis-configured model.
- Verify the alphabet field in pred_gen_opts points to the same alphabet used at training (example_configs/alphabet.npy for the original paper model); mismatched alphabets corrupt token indices.

## Limitations

- Success within 15 predictions is only 69.6% for the full spectrum-to-structure task — sampling is stochastic and the exact molecule is not guaranteed.
- Model was trained on simulated MestreNova-predicted spectra, which do not fully capture nuances of real experimental spectra.
- Accuracy drops sharply for large molecules: below 80% for 18-19 heavy atoms.
- Predictions do not encode stereochemistry; the substructure set and alphabet limit chemical scope.

## Evidence

- [results] top-k random sampling was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] RDKit canonicalization enables exact-match evaluation.: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [readme] README specifies token conventions needed for correct decoding.: [evidence span withheld — anchor status: not_found]
- [readme] README inference entry point command.: "Once the alphabet is set, just run:
```
nmr_infer config.yaml 0 1
```"
- [results] Similarity distribution of incorrect predictions supports Tanimoto-based evaluation.: "89.6% of the best incorrect predictions have a similarity to the correct molecule that is greater than or equal to 0.50, with an average similarity of 0.68."
