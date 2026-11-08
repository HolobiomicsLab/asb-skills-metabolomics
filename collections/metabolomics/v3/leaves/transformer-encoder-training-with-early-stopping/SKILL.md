---
name: transformer-encoder-training-with-early-stopping
description: Use when a task requires regenerating or predicting MS/MS lipid fingerprint
  spectra from lipid structural features (head group, chain length, degree of unsaturation),
  e.g. building a hierarchical spectral library like LipidIN's 5th-level library.
when_to_use_negative:
- The goal is forward annotation of experimental MS2 spectra against an existing library
  (use Flash Entropy Search / annotation module instead of regenerating fingerprints).
- The sample class is PB-derivatized double-bond localization data — those use the
  separate PB library files (PB_PX_H.pkl, common_PX_CH3COO.pkl), not the general corpus.
- Only RT prediction is needed — retention-time mapping is handled by the separate
  LCI intelligence model, not the WMYn.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LipidIN
  role: Framework containing the WMYn network and the general theoretical spectral
    library used as the training corpus for the transformer encoder
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: Spectral Entropy
  role: Entropy-similarity scoring used to validate regenerated fingerprint spectra
    against reference standards
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: FlashEntropySearch
  role: Python Flash entropy search implementation (ms_entropy package) usable to
    compare regenerated spectra against library spectra
  repo: https://github.com/YuanyueLi/FlashEntropySearch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:7011bc832e6a74aae26dd693fe722777a20f48a81e10acb66ee5660050c3b081
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - FlashEntropySearch
  - LipidIN
  - Spectral Entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Transformer encoder training with early stopping

## Summary

Train a multi-head self-attention encoder (the WMYn core) to regenerate lipid fragment fingerprint spectra from structural features, using MSE loss, Adam optimization, and early stopping on validation loss to produce a 5th-level hierarchical library.

## When to use

Use when a task requires regenerating or predicting MS/MS lipid fingerprint spectra from lipid structural features (head group, chain length, degree of unsaturation), e.g. building a hierarchical spectral library like LipidIN's 5th-level library.

## When NOT to use

- The goal is forward annotation of experimental MS2 spectra against an existing library (use Flash Entropy Search / annotation module instead of regenerating fingerprints).
- The sample class is PB-derivatized double-bond localization data — those use the separate PB library files (PB_PX_H.pkl, common_PX_CH3COO.pkl), not the general corpus.
- Only RT prediction is needed — retention-time mapping is handled by the separate LCI intelligence model, not the WMYn.

## Inputs

- LipidIN general theoretical spectral library .pkl files (pos_common.pkl, neg_common.pkl) from Zenodo record 21421866
- Lipid structural feature encodings (head group, chain length, degree of unsaturation of each fatty acyl chain)
- MS2 m/z-intensity sparse matrices as training/validation pairs

## Outputs

- Trained WMYn transformer model weights (PyTorch, transferable pre-trained model independent of sample matrix, instrument, and analytical method)
- Regenerated 5th-level lipid fingerprint spectra library (GeneratedArtifact)
- Validation similarity scores (spectral entropy and cosine similarity against reference spectra)

## How to apply

Encode lipid structural features — head group, chain length, and degree of unsaturation of each fatty acyl chain — into the model input, and load the training corpus of lipid fragmentation fingerprints from the LipidIN general theoretical spectral library (pos_common.pkl / neg_common.pkl from Zenodo record 21421866). Build the WMYn network in PyTorch: a SiLU/ReLU feature extractor projecting the MS2 m/z-intensity sparse matrix into a 512-row latent space, a 6-layer 8-head self-attention encoder with residual connections and layer normalization, then a downsampling–upsampling stage that regenerates fingerprint spectra. Train with MSE loss using the Adam optimizer at learning rate 0.0148 for up to 3000 epochs, applying early stopping on validation loss to prevent overfitting. Use the trained model to regenerate the 5th-level lipid fingerprint library for high-confidence, high-coverage annotation, then validate with spectral entropy / cosine similarity against reference spectra.

## Related tools

- **LipidIN** (Framework containing the WMYn network and the general theoretical spectral library used as the training corpus for the transformer encoder) — https://github.com/LinShuhaiLAB/LipidIN
- **Spectral Entropy** (Entropy-similarity scoring used to validate regenerated fingerprint spectra against reference standards) — https://github.com/YuanyueLi/SpectralEntropy
- **FlashEntropySearch** (Python Flash entropy search implementation (ms_entropy package) usable to compare regenerated spectra against library spectra) — https://github.com/YuanyueLi/FlashEntropySearch

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); es.build_index(spectral_library); sim = es.search(precursor_mz=q['precursor_mz'], peaks=q['peaks'])
```

## Evaluation signals

- Regenerated spectra reach spectral-entropy similarity around 0.9826 against reference standards.
- Cosine similarity >0.96 on NIST SRM 1950 lipids when comparing regenerated vs. experimental spectra.
- Downstream annotation recall improves by approximately +20% (estimated) when using the regenerated 5th-level library.
- Training run shows validation loss plateau/early stop within ≤3000 epochs at Adam learning rate 0.0148 with MSE loss.
- Regenerated spectra cover the hierarchical library structure (all chain compositions and C=C double-bond positions) comparable to the 168.6-million-entry theoretical library.

## Limitations

- The paper notes future work on pre-training on a larger and more diverse dataset to extend reverse lipidomics, implying current transferability of the pre-trained model may not generalize to all matrices.
- Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy can cause feature redundancy when comparing spectra.
- Performance estimates (+20% recall, 0.9826 entropy similarity) are tied to the reported benchmarks (NIST SRM 1950, reference standards) and may not transfer to other cohorts or instruments without retraining.

## Evidence

- [intro] WMYn as reverse lipidomics: "LipidIN incorporates a Wide-spectrum Modeling Yield network (WMYn) as “reverse lipidomics”, which regenerates highly accurate fingerprint spectrograms and easily transferable pre-trained models,"
- [abstract] 20% recall boosting: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [discussion] 5th-level library: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [methods] Training corpus source: "General (exclude PB) | Python `.pkl` (`pos_common.pkl`, `neg_common.pkl`) | 24,631,702"
- [readme] Benchmark metrics: "Theoretical library size | 168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
