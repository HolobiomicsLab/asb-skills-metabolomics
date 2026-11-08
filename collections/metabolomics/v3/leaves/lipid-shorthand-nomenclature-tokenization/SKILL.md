---
name: lipid-shorthand-nomenclature-tokenization
description: Use when labeled lipid species strings (e.g. from HDF5 'lipid_species'
  attributes) must be decomposed into headgroup/fatty-acyl/adduct tokens for seq2seq
  training, per-component accuracy scoring, or token-level prediction comparison.
when_to_use_negative:
- Input is raw profile spectra (mzML) without lipid species labels — tokenization
  only applies to named lipid species, not unlabeled prediction-mode files
- Species names use legacy or divergent nomenclatures that have not been reconciled
  to the updated lipid shorthand notation, since token boundaries will not align with
  the training vocabulary
- The goal is spectral preprocessing (smoothing, baseline correction, peak picking,
  m/z 50–1600 trimming) rather than label/prediction handling
tools:
- name: LipiDetective
  role: Hosts the label tokenization pipeline used to split lipid shorthand names
    into headgroup, fatty acid, and adduct tokens for the transformer decoder and
    for accuracy scoring
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Provides the transformer module whose decoder consumes the tokenized labels
    via per-token embedding vectors
  repo: https://github.com/LipiTUM/lipidetective
- name: h5py
  role: Reads the HDF5 spectra files that carry the lipid_species, adduct, and precursor
    attributes whose strings are tokenized
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:56f13253326ae066cf117be6213d3d9688aeeb81e6e50e1acd7f12607eda2a6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lipid-shorthand-nomenclature-tokenization

## Summary

Splits molecular lipid species names in updated lipid shorthand notation into discrete tokens — headgroup, fatty acid side chains, and adduct — so each token can be embedded as a transformer decoder label (and parsed back from predictions) in LipiDetective.

## When to use

Use when labeled lipid species strings (e.g. from HDF5 'lipid_species' attributes) must be decomposed into headgroup/fatty-acyl/adduct tokens for seq2seq training, per-component accuracy scoring, or token-level prediction comparison.

## When NOT to use

- Input is raw profile spectra (mzML) without lipid species labels — tokenization only applies to named lipid species, not unlabeled prediction-mode files
- Species names use legacy or divergent nomenclatures that have not been reconciled to the updated lipid shorthand notation, since token boundaries will not align with the training vocabulary
- The goal is spectral preprocessing (smoothing, baseline correction, peak picking, m/z 50–1600 trimming) rather than label/prediction handling

## Inputs

- Lipid species name strings in updated lipid shorthand notation (e.g. from lipid_species attributes in HDF5 files under the /all_datasets/ group)
- Predicted lipid species token sequences produced by LipiDetective's greedy/beam decoding
- Token vocabulary/embedding mappings for the transformer decoder labels

## Outputs

- Ordered token sequences (headgroup, fatty acid side chains sorted by carbon count then double-bond count, adduct) of at most 11 tokens per species
- Per-spectrum token-level match vectors enabling lipid component accuracy (correct tokens / total tokens) and total prediction accuracy (all tokens must match)
- Tokenized labels written alongside predictions in test_predictions.csv and scored in test_metrics.csv / test_lipid_metrics.csv

## How to apply

Apply regular-expression matching to split each lipid species name into its headgroup, fatty acid side chains, and adduct, mapping every token to an embedding vector consumed by the transformer decoder (output_seq_length = 11 tokens per species). Sort fatty acid tokens by increasing carbon chain length and then by number of double bonds so the token sequence is canonical regardless of input ordering. Before tokenizing heterogeneous sources, manually reconcile varying nomenclatures to the updated lipid shorthand notation, since token boundaries depend on the shorthand grammar. The same tokenizer must be applied to predicted sequences during test-time scoring: total prediction accuracy counts a prediction correct only if all tokens (headgroup, fatty acid side chains, adduct) match, while lipid component accuracy is computed as correct tokens divided by total tokens. Keep the vocabulary consistent with the model config (e.g. decimal_accuracy 1, max_mz 1600 govern the encoder side, while tokenization governs the label side stored in the HDF5 files).

## Related tools

- **LipiDetective** (Hosts the label tokenization pipeline used to split lipid shorthand names into headgroup, fatty acid, and adduct tokens for the transformer decoder and for accuracy scoring) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Provides the transformer module whose decoder consumes the tokenized labels via per-token embedding vectors) — https://github.com/LipiTUM/lipidetective
- **h5py** (Reads the HDF5 spectra files that carry the lipid_species, adduct, and precursor attributes whose strings are tokenized)

## Examples

```
uv run lipidetective --config config/config_templates/config_transformer.yaml
```

## Evaluation signals

- Every tokenized sequence yields at most output_seq_length = 11 tokens, and the first token is a headgroup class while the last is an adduct
- Fatty acid tokens appear in canonical order: increasing carbon chain length, then increasing number of double bonds
- Round-trip check: rejoining tokens reproduces the original shorthand name, and the adduct token matches the spectrum's polarity attribute (e.g. negative-mode HCE/IOBA-NHC datasets)
- Token-level scoring on the test set reproduces the paper's separation: component accuracy ~60–80% vs total prediction accuracy ~29.7–50.8% on held-out splits
- test_metrics.csv contains per-spectrum component accuracies in [0,1] whose token-weighted aggregate matches the overall component accuracy summary

## Limitations

- Varying nomenclatures across data sources required manual adjustment to the updated lipid shorthand notation; unnormalized names tokenize incorrectly
- Tokenization alone cannot recover unseen species relationships — the model generalizes fragmentation patterns, but a single wrong token makes the total prediction incorrect
- Token count is capped by output_seq_length (11); species names exceeding this budget cannot be represented by the decoder

## Evidence

- [methods] Manual nomenclature normalization precedes tokenization: "The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation"
- [methods] Spectra with lipid species labels are stored in HDF5: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [methods] Transformer consumes tokenized lipid labels from PyTorch module: "LipiDetective was implemented by modifying the transformer module provided by PyTorch"
- [readme] Shorthand-nomenclature output tokens are the model's target format: [evidence span withheld — anchor status: external_doc]
