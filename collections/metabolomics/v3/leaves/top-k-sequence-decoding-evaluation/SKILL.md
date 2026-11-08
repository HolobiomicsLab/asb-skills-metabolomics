---
name: top-k-sequence-decoding-evaluation
description: Use when benchmarking a generative seq2seq model that emits SMILES strings
  (or similar token sequences) and you need top-n exact-match accuracy rather than
  single-greedy-output accuracy.
when_to_use_negative:
- You need the end-to-end spectrum-to-structure accuracy of the multitask model (69.6%
  top-15 from spectra) rather than the substructure-to-structure task upper bound
  (93.2%).
- Inputs are molecules with more than 19 heavy atoms, which are outside the model's
  demonstrated scope.
- You need single deterministic predictions rather than a distribution of sampled
  candidates; greedy decoding with a single output would not measure top-n coverage.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Provides the nmr_infer entry point and token-convention configuration used
    to generate 15 sampled SMILES predictions per input
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: The pretrained substructure-to-structure model whose sampled outputs are evaluated
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes predicted and target SMILES strings and computes heavy-atom
    counts for stratification
- name: Anaconda
  role: Package manager for installing the NMR2Struct environment (environment.yml,
    NMR_env)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:2fdc5ca2486eff07198d7f72032ff2faacf0e195c76ba4cd32101b8bf747cbb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - transformer (encoder-decoder)
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# top-k-sequence-decoding-evaluation

## Summary

Evaluates a generative sequence-to-sequence model (e.g., the NMR2Struct substructure-to-structure transformer) by decoding multiple candidate outputs with top-k random sampling and scoring whether the canonical target appears among the top-n predictions.

## When to use

Use when benchmarking a generative seq2seq model that emits SMILES strings (or similar token sequences) and you need top-n exact-match accuracy rather than single-greedy-output accuracy.

## When NOT to use

- You need the end-to-end spectrum-to-structure accuracy of the multitask model (69.6% top-15 from spectra) rather than the substructure-to-structure task upper bound (93.2%).
- Inputs are molecules with more than 19 heavy atoms, which are outside the model's demonstrated scope.
- You need single deterministic predictions rather than a distribution of sampled candidates; greedy decoding with a single output would not measure top-n coverage.

## Inputs

- Pretrained substructure-to-structure transformer checkpoint (NMR2Struct)
- Substructure label hdf5 files ('substructure_labels' binary vectors) and SMILES targets from Zenodo record 13892026
- Splits file (dictionary of train/validation/test indices) for the 80/10/10 split
- Alphabet token file (checkpoints/alphabet.npy or example_configs/alphabet.npy)

## Outputs

- 15 generated SMILES predictions per test-set input (top-k sampling, k=5)
- Overall top-15 exact-match accuracy and accuracy stratified by heavy-atom count (e.g., stratified_accuracy.csv)

## How to apply

Configure the decoder with top-k random sampling using k = 5 and generate 15 predictions per input, as done in the NMR2Struct paper. Set token conventions before inference: source_size = 957 + 1 = 958 for the 957 substructures, src_pad_token = 0, tgt_pad_token = length of alphabet, tgt_start_token = length of alphabet + 1, tgt_stop_token = length of alphabet + 2, using checkpoints/alphabet.npy (or example_configs/alphabet.npy for the paper's original model). Run nmr_infer over the test set identified by the 80/10/10 splits file from Zenodo record 13892026, then canonicalize both predicted and target SMILES with RDKit and mark a hit when the canonical target SMILES appears among the 15 predictions. Finally, compute overall top-15 accuracy and stratify it by heavy-atom count (e.g., 10, 17, 18-19 heavy atoms) to reveal how accuracy scales with molecular size, writing results such as stratified_accuracy.csv.

## Related tools

- **NMR2Struct** (Provides the nmr_infer entry point and token-convention configuration used to generate 15 sampled SMILES predictions per input) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (The pretrained substructure-to-structure model whose sampled outputs are evaluated) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes predicted and target SMILES strings and computes heavy-atom counts for stratification)
- **Anaconda** (Package manager for installing the NMR2Struct environment (environment.yml, NMR_env))

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- Overall top-15 accuracy on the test set should fall within a few percent of the reported 93.2%.
- Stratified accuracy should be ~96.0% at 10 heavy atoms and ~92.8% at 17 heavy atoms.
- Accuracy for molecules with 18 or 19 heavy atoms should drop below 80%.
- All predicted and target SMILES must canonicalize with RDKit without errors; hits are counted only on canonical canonical-SMILES equality.
- Exactly 15 predictions per input must be generated with top-k sampling set to k = 5.

## Limitations

- Accuracy degrades with molecular size: below 80% for 18-19 heavy atoms, so top-15 sampling coverage is not uniform across the size range.
- The evaluation depends on simulated MestreNova-predicted spectra data and a fixed 80/10/10 random split; no software/package version numbers are stated in the README.
- Sampling is stochastic (top-k random sampling), so exact accuracy can vary run to run unless the seed in global_args is fixed.

## Evidence

- [results] top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] the model achieved an accuracy of 93.2%.: [evidence span withheld — anchor status: not_found]
- [results] at 10 heavy atoms, where the problem size is ∼2 million, the accuracy is 96.0% and only drops to 92.8% by 17 heavy atoms: "at 10 heavy atoms, where the problem size is ∼2 million, the accuracy is 96.0% and only drops to 92.8% by 17 heavy atoms"
- [results] After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit's substructure match functionality: [evidence span withheld — anchor status: not_found]
- [readme] If you want to use the alphabet that was used in the original model from the paper, put the path to the file example_configs/alphabet.npy instead.: "If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy``` instead."
