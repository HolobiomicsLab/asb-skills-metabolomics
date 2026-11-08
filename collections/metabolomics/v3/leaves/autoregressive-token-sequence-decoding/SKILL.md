---
name: autoregressive-token-sequence-decoding
description: Use when reconstructing or extending how a transformer model decodes
  MS2 spectra into lipid shorthand notation strings (headgroup, fatty-acyl chains,
  adduct), or when implementing greedy/beam search decoding with per-prediction confidence
  scores.
when_to_use_negative:
- Input is raw profile spectra that have not yet undergone smoothing, baseline correction,
  peak picking, base peak normalization, and trimming to 50-1600 m/z — decoding requires
  preprocessed spectra
- The goal is spectral-library matching against a fragment database (e.g. LipiDex,
  Alex123) rather than neural-network token decoding
- The task is training or hyperparameter tuning of the transformer (Ray Tune, PyTorch
  Lightning multi-GPU training), not decoding an already-trained model
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model whose linear head (1,032 tokens) is decoded autoregressively
    into lipid shorthand species predictions
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Provides the modified transformer module whose decoder states are mapped through
    the token head during greedy/beam search
- name: PyTorch Lightning
  role: Framework under which the trained model is loaded and executed for autoregressive
    prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:b04023e995e43e6612d0daf473c42320796e1b22fe480b38ca62d462b3c0c8da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# autoregressive-token-sequence-decoding

## Summary

Implements the autoregressive, token-by-token decoding that converts LipiDetective's transformer output into a molecular lipid species prediction, using greedy or beam search over the 1,032-token lipid-name vocabulary until the end-of-sequence token is emitted.

## When to use

Use when reconstructing or extending how a transformer model decodes MS2 spectra into lipid shorthand notation strings (headgroup, fatty-acyl chains, adduct), or when implementing greedy/beam search decoding with per-prediction confidence scores.

## When NOT to use

- Input is raw profile spectra that have not yet undergone smoothing, baseline correction, peak picking, base peak normalization, and trimming to 50-1600 m/z — decoding requires preprocessed spectra
- The goal is spectral-library matching against a fragment database (e.g. LipiDex, Alex123) rather than neural-network token decoding
- The task is training or hyperparameter tuning of the transformer (Ray Tune, PyTorch Lightning multi-GPU training), not decoding an already-trained model

## Inputs

- Pre-trained LipiDetective transformer weights (.pth) with matching training config (d_model, num_heads, n_peaks)
- Preprocessed MS2 spectra for prediction (mzML files, converted to peak-embedding plus positional-encoding input sequences)
- Tokenized lipid-name vocabulary (1,032 output tokens covering headgroup, fatty-acid chains, adduct, end-of-sequence token)

## Outputs

- predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence
- Predicted lipid species strings in shorthand notation with adduct (e.g. 'PC 16:0_18:1 [M+HCOO]-')
- Per-prediction confidence scores filtered at the 0.98 threshold

## How to apply

Load a pre-trained LipiDetective transformer (a .pth checkpoint with the exact training config for d_model, num_heads, and n_peaks) and the preprocessed spectra from the prediction input. Run the transformer encoder over the input sequence of peak embeddings plus positional encodings, built from the top n_peaks m/z values ordered by decreasing intensity. Implement greedy decoding by autoregressively selecting the maximum-probability token at each decoder step from the tokenized lipid-name vocabulary (headgroup, fatty-acid chains, adduct) via a linear head mapping the transformer state to 1,032 output tokens, appending each token to the output sequence and feeding it back as input until the end-of-sequence token is returned. Alternatively implement beam search, retaining the highest cumulative-probability token sequences and returning the best or top-3 complete lipid species strings. Reassemble tokens into lipid shorthand notation with adduct (e.g. 'PC 16:0_18:1 [M+HCOO]-'), compute a confidence score per prediction, apply the 0.98 confidence threshold, and write predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence. Blank-measurement spectra trained the model to emit the end-of-sequence token when no lipid species is present, so an EOS-only output is a valid 'no lipid' result.

## Related tools

- **LipiDetective** (Transformer model whose linear head (1,032 tokens) is decoded autoregressively into lipid shorthand species predictions) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Provides the modified transformer module whose decoder states are mapped through the token head during greedy/beam search)
- **PyTorch Lightning** (Framework under which the trained model is loaded and executed for autoregressive prediction)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- predictions.csv exists and parses as valid CSV containing the prediction and confidence columns
- Every prediction string conforms to the lipid shorthand token format: headgroup + side-chain tokens + adduct (e.g. 'PC 16:0_18:1 [M+HCOO]-')
- Decoding terminates: each spectrum yields either a complete species string or only the end-of-sequence token (blank/no-lipid case)
- Confidence scores are numeric and predictions failing the 0.98 threshold are filtered accordingly
- Beam-search top-3 outputs are complete token sequences with monotonically ordered cumulative probabilities

## Limitations

- Greedy decoding selects the single maximum-probability token per step and may miss higher cumulative-probability sequences that beam search would retain
- Peaks relevant to prediction, such as 224.0 m/z, lack annotation in the Alex123 database, limiting interpretability of decoded components
- The vocabulary is fixed at 1,032 tokens from training; lipid species never seen during training can still be predicted via generalization, but tokens outside the vocabulary cannot be emitted

## Evidence

- [methods] autoregressive decoding via linear head and EOS: [evidence span withheld — anchor status: not_found]
- [results] EOS on blanks trained explicitly: "2002 MS2 spectra from blank measurements that do not show any target lipid species were added"
- [readme] README prediction workflow: [evidence span withheld — anchor status: not_found]
