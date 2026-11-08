---
name: heavy-atom-count-stratified-model-evaluation
description: Use when a structure-generation model (e.g., a transformer producing
  SMILES) has been run over a test set and you need to quantify exact-match accuracy
  both overall and per molecular size class (heavy-atom count) to detect size-dependent
  degradation.
when_to_use_negative:
- When evaluating the spectrum-to-structure multitask model end-to-end from 1H/13C
  NMR spectra — this skill isolates the substructure-to-structure stage only
- When the evaluation goal is structural similarity (Tanimoto similarity of incorrect
  predictions) rather than exact canonical-SMILES match
- When predictions come from a different sampling protocol than top-k random sampling
  with k=5 and 15 predictions per input, since the reported accuracy baselines are
  protocol-specific
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Canonicalize predicted and target SMILES and determine heavy-atom counts before
    the exact-match hit calculation
- name: NMR2Struct
  role: Runs the transformer inference (nmr_infer) that generates the 15 SMILES predictions
    per test input used for stratified scoring
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: The substructure-to-structure model being evaluated; decodes SMILES from the
    957-substructure arrays via top-k sampling (k=5)
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: Anaconda
  role: Package manager for installing the NMR2Struct environment (environment.yml,
    NMR_env) in which inference and evaluation run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:2d5cb2be722930bc6d23293360e29d00b1f6f67c9f3ac798326f4f0c955de7a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - transformer (encoder-decoder)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Heavy-atom-count-stratified model evaluation

## Summary

Evaluates a molecular structure generation model by computing overall top-k hit accuracy and breaking it down by the heavy-atom count of target molecules, revealing how reconstruction accuracy degrades as molecular size and combinatorial search space grow.

## When to use

Use when a structure-generation model (e.g., a transformer producing SMILES) has been run over a test set and you need to quantify exact-match accuracy both overall and per molecular size class (heavy-atom count) to detect size-dependent degradation.

## When NOT to use

- When evaluating the spectrum-to-structure multitask model end-to-end from 1H/13C NMR spectra — this skill isolates the substructure-to-structure stage only
- When the evaluation goal is structural similarity (Tanimoto similarity of incorrect predictions) rather than exact canonical-SMILES match
- When predictions come from a different sampling protocol than top-k random sampling with k=5 and 15 predictions per input, since the reported accuracy baselines are protocol-specific

## Inputs

- Test-set split indices (splits file with train/val/test indices, or 80/10/10 fractions)
- Substructure label hdf5 files with 'substructure_labels' binary vectors per molecule
- SMILES target numpy file (binary-encoded strings, decode with x.decode('utf-8'))
- Model-generated SMILES predictions (15 per input, from top-k sampling with k=5)
- Pretrained substructure-to-structure transformer checkpoint and alphabet (checkpoints/alphabet.npy or example_configs/alphabet.npy)

## Outputs

- Overall top-15 exact-match accuracy on the canonical-SMILES test set (e.g., ~93.2%)
- Accuracy stratified by heavy-atom count bins (10, 17, 18-19 heavy atoms), e.g., stratified_accuracy.csv
- Per-molecule heavy-atom counts and per-molecule hit/no-hit labels

## How to apply

Run inference over the test set (from the paper's 80/10/10 random split) using top-k random sampling with k=5 to generate 15 predictions per input, matching the paper's protocol. Canonicalize both predicted and target SMILES with RDKit, then mark a hit when the canonical target SMILES appears among the 15 predictions for a molecule. Compute each target molecule's heavy-atom (non-hydrogen) count from its structure. Aggregate overall accuracy and stratified accuracy at size bins used in the paper (10 heavy atoms, 17 heavy atoms, and 18-19 heavy atoms), writing results to a file such as stratified_accuracy.csv and comparing against reported benchmarks (93.2% overall, 96.0% at 10, 92.8% at 17, below 80% at 18-19). The rationale is that the number of possible structures grows combinatorially with heavy atoms, so a single overall accuracy number hides meaningful size-dependent failure modes.

## Related tools

- **RDKit** (Canonicalize predicted and target SMILES and determine heavy-atom counts before the exact-match hit calculation)
- **NMR2Struct** (Runs the transformer inference (nmr_infer) that generates the 15 SMILES predictions per test input used for stratified scoring) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (The substructure-to-structure model being evaluated; decodes SMILES from the 957-substructure arrays via top-k sampling (k=5)) — https://github.com/MarklandGroup/NMR2Struct
- **Anaconda** (Package manager for installing the NMR2Struct environment (environment.yml, NMR_env) in which inference and evaluation run)

## Examples

```
nmr_infer config.yaml 0 1  # generate 15 SMILES per test input (top-k, k=5), then canonicalize with RDKit and compute hit rate stratified by heavy-atom count
```

## Evaluation signals

- Overall accuracy reproduces the reported 93.2% top-15 accuracy on the test set (within a few percent)
- Stratified accuracies follow the reported trend: 96.0% at 10 heavy atoms, 92.8% at 17 heavy atoms, and below 80% at 18-19 heavy atoms
- Accuracy is monotonically non-increasing with heavy-atom count, reflecting the combinatorial growth of possible structures
- All predicted and target SMILES are canonicalized with the same RDKit version/procedure before string comparison, so no false negatives arise from non-canonical SMILES
- Every test-set molecule yields exactly 15 generated predictions before hit evaluation

## Limitations

- Accuracy drops below 80% for molecules with 18 or 19 heavy atoms, so stratified metrics at the largest sizes are high-variance and close to the model's regime of failure
- The framework is demonstrated only on molecules with up to 19 heavy atoms containing only C, N, and O as non-hydrogen atoms; extrapolating stratified metrics beyond this domain is unsupported
- Top-k random sampling introduces sampling variance, so accuracy estimates depend on the stochastic generation run

## Evidence

- [results] Substructure-to-structure accuracy was 96.0% at 10 heavy atoms and 92.8% at 17 heavy atoms: "at 10 heavy atoms, where the problem size is ∼2 million, the accuracy is 96.0% and only drops to 92.8% by 17 heavy atoms"
- [results] Substructure-to-structure accuracy dropped below 80% for molecules with 18 or 19 heavy atoms: "the accuracy drops below 80% for the molecules with 18 or 19 heavy atoms"
- [results] Evaluation protocol: top-k sampling with k=5, 15 predictions per input, accuracy by canonical SMILES match: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] RDKit used for SMILES canonicalization and substructure matching: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [readme] Inference entry point that produces the predictions scored by this skill: "Once the alphabet is set, just run:
```
nmr_infer config.yaml 0 1
```"
- [results] Overall reported accuracy benchmark for the substructure-to-structure transformer: "the model achieved an accuracy of 93.2%."
