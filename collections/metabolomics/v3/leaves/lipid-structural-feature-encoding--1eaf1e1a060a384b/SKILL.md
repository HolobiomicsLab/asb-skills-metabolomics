---
name: lipid-structural-feature-encoding
description: 'Use when preparing structured inputs for the WMYn reverse-lipidomics
  model: lipid annotations (head group, chain length, degree of unsaturation per fatty
  acyl chain) must be converted into the feature representation the network consumes
  before MS2 fingerprint regeneration.'
when_to_use_negative:
- When the goal is annotating already-acquired MS2 spectra by similarity search against
  an existing library — use Flash Entropy Search / MS-DIAL style search instead of
  reverse-lipidomics encoding.
- When the task is double-bond localization via Paternò–Büchi derivatization, which
  relies on PB reaction spectra rather than structural-feature encoding.
- When input lipids lack resolvable chain composition/unsaturation information, since
  the encoding is defined over head group, chain length, and per-chain degree of unsaturation.
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: LipidIN
  role: Framework whose WMYn consumes the encoded lipid structural features to regenerate
    fingerprint spectra as the 5th-level library; training corpus loaded from its
    Zenodo .pkl libraries.
  repo: github.com/LinShuhaiLAB/LipidIN
- name: PyTorch
  role: Implementation backend for the WMYn SiLU/ReLU feature extractor, 6-layer 8-head
    self-attention encoder, and Adam (lr 0.0148) MSE training loop.
- name: Spectral Entropy
  role: Used downstream to compute spectral-entropy similarity of regenerated spectra
    against reference standards for validation of the encoded-feature model output.
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:669b59c93eeef4369867937b14ffe02b58972b6bc398e13f5189eb57baaa2ed2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - PyTorch
  - Spectral Entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lipid-structural-feature-encoding

## Summary

Encodes lipid structural features — head group, fatty-acyl chain length, and degree of unsaturation of each chain — into the numeric input representation required by the WMYn (Wide-spectrum Modeling Yield network) for reverse-lipidomics fingerprint regeneration in LipidIN.

## When to use

Use when preparing structured inputs for the WMYn reverse-lipidomics model: lipid annotations (head group, chain length, degree of unsaturation per fatty acyl chain) must be converted into the feature representation the network consumes before MS2 fingerprint regeneration.

## When NOT to use

- When the goal is annotating already-acquired MS2 spectra by similarity search against an existing library — use Flash Entropy Search / MS-DIAL style search instead of reverse-lipidomics encoding.
- When the task is double-bond localization via Paternò–Büchi derivatization, which relies on PB reaction spectra rather than structural-feature encoding.
- When input lipids lack resolvable chain composition/unsaturation information, since the encoding is defined over head group, chain length, and per-chain degree of unsaturation.

## Inputs

- LipidIN general theoretical spectral library pickle files (`pos_common.pkl`, `neg_common.pkl`, 24,631,702 spectra; Zenodo record 21421866)
- Lipid structural annotations: head group, fatty-acyl chain length, and degree of unsaturation per chain (and C=C double-bond positions) from the 168.6-million-entry theoretical library
- Optional PB-library spectra (`PB_PX_H.pkl`, `common_PX_CH3COO.pkl` + MS1 index, 2,289,913 spectra) for Paternò–Büchi workflows

## Outputs

- Encoded structural-feature input vectors consumed by the WMYn network (SiLU/ReLU extractor → 512-row latent space)
- Trained/transferable WMYn pre-trained model
- Regenerated lipid fragment fingerprint spectra constituting the 5th-level hierarchical library

## How to apply

First assemble the training corpus from the LipidIN general theoretical spectral library (`pos_common.pkl` / `neg_common.pkl`, 24,631,702 spectra, Zenodo record 21421866), whose entries carry chain compositions and C=C double-bond positions from the 168.6-million-entry hierarchical library. Encode each lipid's structural features — head group, chain length, and degree of unsaturation of each fatty acyl chain — into the WMYn input representation. Feed the encoded representation into the WMYn pipeline: a SiLU/ReLU feature extractor projecting into a 512-row latent space, then a 6-layer, 8-head self-attention encoder with residual connections and layer normalization, followed by downsampling-upsampling regeneration of the fingerprint spectra. Train with MSE loss using the Adam optimizer at learning rate 0.0148 for up to 3000 epochs, applying early stopping on validation loss. The rationale (per the article) is that deep learning can extract the mapping between lipid structure features and fragmentation behavior, so the encoding quality directly determines whether the regenerated 5th-level library spectra reach the reported similarity thresholds.

## Related tools

- **LipidIN** (Framework whose WMYn consumes the encoded lipid structural features to regenerate fingerprint spectra as the 5th-level library; training corpus loaded from its Zenodo .pkl libraries.) — github.com/LinShuhaiLAB/LipidIN
- **PyTorch** (Implementation backend for the WMYn SiLU/ReLU feature extractor, 6-layer 8-head self-attention encoder, and Adam (lr 0.0148) MSE training loop.)
- **Spectral Entropy** (Used downstream to compute spectral-entropy similarity of regenerated spectra against reference standards for validation of the encoded-feature model output.) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
import pandas as pd; lib = pd.read_pickle('pos_common.pkl')  # LipidIN general library, Zenodo record 21421866
```

## Evaluation signals

- Encoded features must cover exactly the three structural axes the paper names: head group, chain length, and degree of unsaturation of each fatty acyl chain.
- Encoded inputs must project through the WMYn extractor into a 512-row latent space before the 6-layer/8-head self-attention encoder.
- Regenerated spectra from the encoded inputs must reach spectral-entropy similarity around 0.9826 against reference standards.
- Cosine similarity on NIST SRM 1950 lipids must exceed 0.96.
- Downstream annotation recall should improve by approximately +20% (the reported WMYn estimated recall boost).

## Limitations

- The article flags pre-training on a larger and more diverse dataset as a stated future need for extending reverse lipidomics, so current encodings/models may not generalize to all matrices.
- The WMYn is presented as 'reverse lipidomics' with transferable pre-trained models independent of sample matrix, instrument, and analytical method, but validation cited is limited to reference standards and NIST SRM 1950 lipids.
- The regenerated library is a theoretical construct (5th-level library); its utility depends on the quality of the underlying 168.6-million-entry chain-composition/C=C-position theoretical library.

## Evidence

- [intro] We hypothesized that deep learning models could extract the mapping relationship between lipid structure features (e.g., head group, chain length, and degree of unsaturation of each fatty acyl or: "We hypothesized that deep learning models could extract the mapping relationship between lipid structure features (e.g., head group, chain length, and degree of unsaturation of each fatty acyl or"
- [abstract] LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting.: "LipidIN integrates a Wide-spectrum Modeling Yield network for regenerating lipid fragment fingerprints to further improve accuracy and coverage with a 20% estimated recall boosting."
- [methods] General (exclude PB) | Python `.pkl` (`pos_common.pkl`, `neg_common.pkl`) | 24,631,702: "General (exclude PB) | Python `.pkl` (`pos_common.pkl`, `neg_common.pkl`) | 24,631,702"
- [readme] WMYn recall change | +20 % (estimated): "WMYn recall change | +20 % (estimated)"
- [discussion] regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
