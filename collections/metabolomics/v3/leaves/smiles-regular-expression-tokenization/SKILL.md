---
name: smiles-regular-expression-tokenization
description: 'Use when training or running the NMR2Struct transformer: canonical SMILES
  targets must be converted into integer token sequences with a consistent alphabet
  and start/stop/pad conventions before teacher-forced training or top-k sampling
  inference.'
when_to_use_negative:
- Input is a 1H/13C NMR spectrum for the multitask model's encoder inputs — those
  are interpolated spectra (1H grid of 28000 values) or 80-binned 13C vectors, not
  SMILES token sequences
- The substructure input side of the substructure-to-structure transformer — that
  uses the compressed integer index list (1 to 957) of present substructures, not
  regex tokenization
- You already have token sequences in a compatible pretrained alphabet and only need
  to run inference — just load the existing alphabet.npy and set the token-convention
  fields
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Canonicalizes SMILES strings before tokenization and canonicalizes decoded
    SMILES predictions for exact-match comparison
- name: NMR2Struct
  role: Provides the regex tokenizer, alphabet file (example_configs/alphabet.npy),
    and YAML token-convention fields (source_size, tgt_pad_token, tgt_start_token,
    tgt_stop_token)
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer (encoder-decoder)
  role: Consumes the tokenized target SMILES sequences autoregressively during training
    and top-k (k=5) SMILES generation
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:e1cb480ddc1da8cf2c4a156a2b14353a9bff04d7885429f78dedccd38d01b839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - transformer (encoder-decoder)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# SMILES regular-expression tokenization

## Summary

Tokenizes canonicalized SMILES strings with a regular-expression tokenizer and builds a token alphabet from unique tokens across the dataset, enabling autoregressive SMILES generation in the NMR2Struct substructure-to-structure transformer.

## When to use

Use when training or running the NMR2Struct transformer: canonical SMILES targets must be converted into integer token sequences with a consistent alphabet and start/stop/pad conventions before teacher-forced training or top-k sampling inference.

## When NOT to use

- Input is a 1H/13C NMR spectrum for the multitask model's encoder inputs — those are interpolated spectra (1H grid of 28000 values) or 80-binned 13C vectors, not SMILES token sequences
- The substructure input side of the substructure-to-structure transformer — that uses the compressed integer index list (1 to 957) of present substructures, not regex tokenization
- You already have token sequences in a compatible pretrained alphabet and only need to run inference — just load the existing alphabet.npy and set the token-convention fields

## Inputs

- RDKit-canonicalized SMILES strings (e.g. from the ~3.1 million molecule SpectraBase + GDB-17 training set, stored as a numpy .npy file with binary-encoded strings)
- Existing token alphabet .npy file (checkpoints/alphabet.npy or example_configs/alphabet.npy) when reusing a pretrained transformer

## Outputs

- Token alphabet mapping unique SMILES tokens to integer IDs (alphabet.npy)
- Integer token sequences per molecule with right padding (pad = len(alphabet), start = len(alphabet)+1, stop = len(alphabet)+2)
- Decoded SMILES strings produced at inference, passed to RDKit for canonicalization and exact-match evaluation

## How to apply

After canonicalizing all SMILES with RDKit, tokenize each canonical SMILES using a regular-expression tokenizer so that atoms, branches, rings, and stereo/charge symbols become single tokens; then build the alphabet from all unique tokens across the dataset. Define the special-token scheme by the NMR2Struct convention: target_size = length of alphabet + 3, tgt_pad_token = length of alphabet, tgt_start_token = length of alphabet + 1, tgt_stop_token = length of alphabet + 2 (e.g. alphabet.npy in example_configs stores the original paper's alphabet). During batching, apply right padding to SMILES vectors and mask the padding tokens in the loss. The same alphabet must be reused at inference (and when initializing the multitask spectrum-to-structure model from pretrained transformer weights) so decoded token IDs map back to the correct characters for RDKit canonicalization and exact-match scoring.

## Related tools

- **RDKit** (Canonicalizes SMILES strings before tokenization and canonicalizes decoded SMILES predictions for exact-match comparison)
- **NMR2Struct** (Provides the regex tokenizer, alphabet file (example_configs/alphabet.npy), and YAML token-convention fields (source_size, tgt_pad_token, tgt_start_token, tgt_stop_token)) — https://github.com/MarklandGroup/NMR2Struct
- **transformer (encoder-decoder)** (Consumes the tokenized target SMILES sequences autoregressively during training and top-k (k=5) SMILES generation) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
alphabet = np.load('example_configs/alphabet.npy', allow_pickle=True); tgt_pad, tgt_start, tgt_stop = len(alphabet), len(alphabet)+1, len(alphabet)+2
```

## Evaluation signals

- model.model_args.target_size equals length of alphabet + 3 and tgt_pad_token equals length of alphabet, per the README token-convention table
- Every canonical SMILES in the dataset tokenizes without out-of-alphabet tokens; reconstructing a token sequence yields a valid SMILES that RDKit canonicalizes to the original string
- Padding tokens (id = len(alphabet)) are masked during training, matching the paper's 'right padding with masking' scheme
- Using the paper's original alphabet (example_configs/alphabet.npy) and the pretrained weights, reproduction reaches the benchmark 93.2% substructure-to-structure accuracy with 15 top-k predictions per input
- Alphabet is consistent between training and inference (data.alphabet set to the pretrained transformer's alphabet when initializing the multitask model)

## Limitations

- Token-convention fields must be set manually when inferring without retraining; wrong tgt_start_token/tgt_stop_token values silently break generation
- SMILES stored as a numpy binary array must be decoded with x.decode('utf-8') before tokenization
- The alphabet from training must be reused at inference; mismatching alphabets invalidate pretrained weight transfer to the multitask model
- Vocabulary is fixed to the unique tokens of the training set, so SMILES containing unseen symbols (e.g. stereochemistry notation, other elements) cannot be represented — the model itself does not yet predict stereochemistry

## Evidence

- [results] Tokenized canonicalized SMILES using a regular expression and determined an alphabet from unique tokens: "the canonicalized SMILES were tokenized using a regular expression,32 and an alphabet was determined from all unique tokens across all SMILES"
- [readme] Token convention table defining pad/start/stop relative to alphabet length: "model.model_args.target_size | length of alphabet + 3"
- [readme] Alphabet consistency with the pretrained model: "If you want to use the alphabet that was used in the original model from the paper, put the path to the file ```example_configs/alphabet.npy``` instead."
- [results] Right padding and masking during training: "Right padding was applied to ensure that all substructure vectors and SMILES vectors in a batch were of the same length"
