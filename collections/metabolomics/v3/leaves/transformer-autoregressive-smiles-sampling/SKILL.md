---
name: transformer-autoregressive-smiles-sampling
description: Use when you have binary substructure label vectors (957 SMART-derived
  substructures, hdf5 format) for a molecule with ≤19 heavy atoms and need candidate
  SMILES reconstructions scored by top-15 exact-match accuracy.
when_to_use_negative:
- The input is a 1H/13C NMR spectrum rather than a precomputed substructure array
  — use the multitask spectrum-to-structure model or nmr_infer_single_spectrum instead,
  as this entry point only works for substructure-to-structure inference.
- The molecule exceeds 19 heavy atoms or requires stereochemistry — accuracy drops
  below 80% at 18-19 heavy atoms and the model does not predict stereochemistry.
- You need deterministic single-structure output rather than candidate sampling —
  this protocol inherently generates 15 stochastic samples per input via top-k sampling.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Package providing the pretrained substructure-to-structure transformer checkpoint,
    token conventions, alphabet file, and nmr_infer entry point
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: Decoder autoregressively generates SMILES tokens from embedded present-substructure
    indices; pretrained on the substructure-to-structure task
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes generated and target SMILES strings for exact-match scoring
    and computing heavy-atom counts and Tanimoto similarity of incorrect predictions
- name: Anaconda
  role: Package manager used to create the NMR_env environment from environment.yml
    before running inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:e71c4dfc265ae4335ff4b24f8aa687db055024947f039a4b876d30f0c12b9a72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - transformer
  - transformer (encoder-decoder)
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# transformer-autoregressive-smiles-sampling

## Summary

Run the pretrained substructure-to-structure encoder-decoder transformer from NMR2Struct to autoregressively generate candidate SMILES strings from a 957-element binary substructure array, using top-k random sampling (k=5) with 15 predictions per input to reconstruct exact molecular structures.

## When to use

Use when you have binary substructure label vectors (957 SMART-derived substructures, hdf5 format) for a molecule with ≤19 heavy atoms and need candidate SMILES reconstructions scored by top-15 exact-match accuracy.

## When NOT to use

- The input is a 1H/13C NMR spectrum rather than a precomputed substructure array — use the multitask spectrum-to-structure model or nmr_infer_single_spectrum instead, as this entry point only works for substructure-to-structure inference.
- The molecule exceeds 19 heavy atoms or requires stereochemistry — accuracy drops below 80% at 18-19 heavy atoms and the model does not predict stereochemistry.
- You need deterministic single-structure output rather than candidate sampling — this protocol inherently generates 15 stochastic samples per input via top-k sampling.

## Inputs

- hdf5 substructure label files containing the 'substructure_labels' dataset (one binary 957-element vector per molecule)
- numpy-encoded SMILES target strings (x.decode('utf-8') required when reading)
- Pretrained substructure-to-structure transformer checkpoint
- alphabet.npy token alphabet (checkpoints/alphabet.npy or example_configs/alphabet.npy)
- YAML config with correctly populated model, inference, and token-convention fields

## Outputs

- Raw generated SMILES predictions per input (hdf5 file written to the global_args save directory)
- Top-15 exact-match accuracy, optionally stratified by heavy-atom count (e.g., stratified_accuracy.csv)

## How to apply

Load the pretrained checkpoint and set token conventions per the README: source_size = 957+1 = 958, src_pad_token = 0, tgt_pad_token = length of alphabet, tgt_start_token = length of alphabet+1, tgt_stop_token = length of alphabet+2, with alphabet from checkpoints/alphabet.npy (or example_configs/alphabet.npy for the original paper model). Substructure arrays are presented as integer index lists (1 to 957) of the present (nonzero) substructures, right-padded and masked. Run nmr_infer on the test split so the decoder autoregressively emits SMILES tokens until the stop token; use top-k random sampling with k=5 and generate 15 predictions per input, matching the paper protocol. Canonicalize predicted and target SMILES with RDKit and score a hit when the canonical target SMILES appears among the 15 predictions.

## Related tools

- **NMR2Struct** (Package providing the pretrained substructure-to-structure transformer checkpoint, token conventions, alphabet file, and nmr_infer entry point) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (Decoder autoregressively generates SMILES tokens from embedded present-substructure indices; pretrained on the substructure-to-structure task) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes generated and target SMILES strings for exact-match scoring and computing heavy-atom counts and Tanimoto similarity of incorrect predictions)
- **Anaconda** (Package manager used to create the NMR_env environment from environment.yml before running inference)

## Examples

```
nmr_infer config.yaml 0 1
```

## Evaluation signals

- Overall top-15 accuracy on the 80/10/10 test split should reproduce ~93.2% (top-k sampling, k=5, 15 predictions per input).
- Stratified accuracy by heavy-atom count should show 96.0% at 10 heavy atoms, 92.8% at 17, and below 80% at 18-19.
- Token fields must satisfy: source_size=958, tgt_pad_token=len(alphabet), tgt_start_token=len(alphabet)+1, tgt_stop_token=len(alphabet)+2.
- All predicted and target SMILES canonicalize successfully with RDKit; ~89.6% of best incorrect predictions should have Tanimoto similarity ≥0.50 (average ~0.68).
- Generated strings terminate at tgt_stop_token and use tokens drawn from alphabet.npy.

## Limitations

- Accuracy drops below 80% for molecules with 18 or 19 heavy atoms, where the combinatorial space reaches trillions of structures.
- The decoder does not emit stereochemistry; enabling it would require expanding the 957-substructure set with larger, more complicated chemical motifs.
- Generating 15 stochastic predictions via top-k sampling (k=5) introduces run-to-run variance; exact reproduction requires matching seed and inference config.
- The pretrained checkpoint is only valid with its own alphabet/token layout — use example_configs/alphabet.npy, not a newly trained alphabet, to reproduce paper numbers.

## Evidence

- [results] top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] the model achieved an accuracy of 93.2%.: [evidence span withheld — anchor status: not_found]
- [results] an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule.: "an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule."
- [readme] source_size = 957 + 1 (e.g., 958 for the 957 substructures): "| ```model.model_args.source_size``` | number of substructures + 1 (e.g., 958 for the 957 substructures) |"
- [readme] If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy```: "If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy``` instead."
- [readme] Once the alphabet is set, just run: nmr_infer config.yaml 0 1: "Once the alphabet is set, just run:
```
nmr_infer config.yaml 0 1
```"
