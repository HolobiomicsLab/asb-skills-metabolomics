---
name: beam-search-over-attention-decoder
description: Use when converting a trained transformer's MS2-spectrum representation
  into a molecular lipid species prediction requires autoregressive token-by-token
  decoding over a fixed lipid-name vocabulary, with greedy or beam-search alternatives.
when_to_use_negative:
- The input is raw profile data that has not yet been smoothed, baseline-corrected,
  peak-picked, base-peak normalized, and trimmed to 50–1600 m/z — run the preprocessing/filtering
  skill first.
- You need to train the transformer or tune hyperparameters (Ray Tune) rather than
  decode predictions from an already-trained checkpoint.
- The task is fragment annotation or interpretability analysis (integrated gradients
  via Captum) rather than converting a decoder state into a lipid species string.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: LipiDetective
  role: Transformer model whose linear head decodes into the 1,032-token lipid-shorthand
    vocabulary; this skill implements its greedy/beam decoding loop
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Framework whose modified transformer module hosts the encoder and autoregressive
    decoder loop
- name: PyTorch Lightning
  role: Training/infra wrapper around the transformer; the decoding step consumes
    checkpoints produced through it
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:7a7fff2be2802609f7fe015d544a9fd6e174112e3aabb7f632d5e72f99a4da8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# beam-search-over-attention-decoder

## Summary

Implements the token-level decoding stage of LipiDetective's transformer: the encoder representation is mapped through a linear head onto a 1,032-token lipid-shorthand vocabulary and decoded autoregressively (greedy or beam search) until the end-of-sequence token, yielding lipid species strings with adduct and a confidence score.

## When to use

Use when converting a trained transformer's MS2-spectrum representation into a molecular lipid species prediction requires autoregressive token-by-token decoding over a fixed lipid-name vocabulary, with greedy or beam-search alternatives.

## When NOT to use

- The input is raw profile data that has not yet been smoothed, baseline-corrected, peak-picked, base-peak normalized, and trimmed to 50–1600 m/z — run the preprocessing/filtering skill first.
- You need to train the transformer or tune hyperparameters (Ray Tune) rather than decode predictions from an already-trained checkpoint.
- The task is fragment annotation or interpretability analysis (integrated gradients via Captum) rather than converting a decoder state into a lipid species string.

## Inputs

- Pre-trained LipiDetective transformer weights in .pth format with matching training config (d_model, num_heads, n_peaks)
- Preprocessed MS2 spectra (top n_peaks m/z values ordered by decreasing intensity, base-peak normalized, trimmed to 50–1600 m/z)
- Output token vocabulary of 1,032 lipid-shorthand tokens (headgroup, fatty-acid chains, adduct, end-of-sequence)

## Outputs

- Decoded lipid species strings in shorthand notation with adduct, e.g. 'PC 16:0_18:1 [M+HCOO]-' (greedy or best/top-3 beam sequences)
- Per-prediction confidence score used against the 0.98 threshold
- predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, confidence

## How to apply

Load the pre-trained LipiDetective transformer weights (.pth) together with the exact training configuration for d_model, num_heads, and n_peaks. Run the transformer encoder over the input sequence of top n_peaks m/z values ordered by decreasing intensity, where each peak embedding carries positional encoding. Attach a linear head that maps the decoder state onto the 1,032-token vocabulary covering headgroup, fatty-acid side-chain, and adduct tokens; for greedy decoding select the argmax token at each step, append it to the output sequence, and feed it back as input until the end-of-sequence token is emitted (the same token the model returns for blank spectra where no lipid species is present). For beam search retain the top-K token sequences by cumulative probability and return the best (or top-3) complete lipid strings. Reassemble tokens into lipid shorthand notation (e.g. 'PC 16:0_18:1 [M+HCOO]-'), compute the per-prediction confidence, and drop predictions below the 0.98 threshold before writing predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence.

## Related tools

- **LipiDetective** (Transformer model whose linear head decodes into the 1,032-token lipid-shorthand vocabulary; this skill implements its greedy/beam decoding loop) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Framework whose modified transformer module hosts the encoder and autoregressive decoder loop)
- **PyTorch Lightning** (Training/infra wrapper around the transformer; the decoding step consumes checkpoints produced through it)

## Examples

```
uv run lipidetective --config config/config_templates/prediction.yaml
```

## Evaluation signals

- predictions.csv exists, parses as valid CSV, and contains the file, polarity, spectrum_index, precursor, prediction, and confidence columns
- Every emitted prediction string conforms to the lipid shorthand token grammar: headgroup + fatty-acid side-chain tokens + adduct token, terminated by the end-of-sequence token
- The number of decoded tokens per spectrum is finite — decoding terminates on the end-of-sequence token, never loops indefinitely
- Beam search agrees with greedy decoding on the top-1 lipid species for the same input spectrum (or the disagreement is explained by low confidence)
- All retained predictions have confidence ≥ 0.98; sub-threshold predictions are filtered before writing results

## Limitations

- Decoding vocabulary is fixed at 1,032 tokens, so lipid species outside the learned shorthand vocabulary cannot be emitted
- MS2 spectra from blank measurements (2,002 spectra) were added so the model emits the end-of-sequence token when no lipid species is present — pure-noise inputs may legitimately produce an empty prediction
- The linear-head argmax decoder is only as good as the generalization of learned fragmentation patterns; the article notes the model correctly predicts unseen species but reproducibility across labs depends on consistent preprocessing upstream
- Peak relevance can depend on fragments absent from the Alex123 annotation database (e.g. 224.0 m/z), so decoded strings are not always accompanied by a database fragment annotation

## Evidence

- [results] Autoregressive greedy decoding to end-of-sequence token over 1,032-token vocabulary: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing context: m/z trim range for MS2 input spectra: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [results] Blank spectra teach end-of-sequence emission when no lipid species present: "2002 MS2 spectra from blank measurements that do not show any target lipid species were added"
- [readme] README: prediction from mzML via YAML config: [evidence span withheld — anchor status: external_doc]
- [readme] README: command-line entry point for running the model: "uv run lipidetective --config path/to/config.yaml"
