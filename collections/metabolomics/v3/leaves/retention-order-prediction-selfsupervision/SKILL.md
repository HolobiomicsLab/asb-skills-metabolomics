---
name: retention-order-prediction-selfsupervision
description: Use when training MS/MS spectral encoders on large unannotated LC-MS/MS
  collections (e.g., GeMS mined from MassIVE GNPS) and you want representations that
  cluster by molecular structure without supervised labels.
when_to_use_negative:
- When you have a labeled spectral library (MoNA, NIST20) and a supervised prediction
  objective — fine-tuned DreaMS or MIST-style supervised models are the appropriate
  choice there
- When your spectra are outside the training distribution, e.g., negative ionization
  mode or multiply charged precursor ions, which the current model does not incorporate
- When the analysis requires MS1 isotopic patterns or adduct distributions — this
  method focuses solely on tandem mass spectra
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: pluskal-lab/DreaMS
  role: Reference implementation of the transformer architecture, self-supervised
    pre-training (masked m/z prediction + retention order prediction), and embedding
    extraction
  repo: github.com__pluskal-lab__DreaMS
- name: HDF5
  role: Compact binary storage format for GeMS spectra and LC-MS/MS metadata feeding
    the pre-training pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:e6ba17990446136e68d9eb8e03cf6678282f2e84242f141a135d7a74e1070ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  - pluskal-lab/DreaMS
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Retention-order-prediction self-supervision for MS/MS representation learning

## Summary

A self-supervised pre-training scheme in which a 116-million-parameter transformer (DreaMS) is trained on unannotated tandem mass spectra from the GeMS dataset using masked m/z-bin peak prediction and chromatographic retention order prediction as auxiliary tasks. This induces 1,024-dimensional embeddings organized by molecular structural similarity without requiring labeled spectra.

## When to use

Use when training MS/MS spectral encoders on large unannotated LC-MS/MS collections (e.g., GeMS mined from MassIVE GNPS) and you want representations that cluster by molecular structure without supervised labels.

## When NOT to use

- When you have a labeled spectral library (MoNA, NIST20) and a supervised prediction objective — fine-tuned DreaMS or MIST-style supervised models are the appropriate choice there
- When your spectra are outside the training distribution, e.g., negative ionization mode or multiply charged precursor ions, which the current model does not incorporate
- When the analysis requires MS1 isotopic patterns or adduct distributions — this method focuses solely on tandem mass spectra

## Inputs

- GeMS dataset (HDF5-based binary format) of unannotated MS/MS spectra with selected LC-MS/MS metadata, mined from the MassIVE GNPS repository
- Retention order pairs (spectra ordered along chromatographic gradients) for the retention order prediction task
- Held-out spectra grouped by 2D InChI key for clustering evaluation
- MACCS molecular fingerprint labels for linear probing

## Outputs

- Pre-trained DreaMS transformer checkpoint (116M parameters)
- 1,024-dimensional real-valued embeddings per held-out spectrum
- k=3 linear clustering accuracy in the 1,023-D PCA subspace (target ~0.865)
- Per-depth linear-probe precision for MACCS fingerprint bits (0.81-0.84 regime)

## How to apply

First prepare spectra in the compact HDF5-based binary format the authors designed for deep learning, after mining and quality-filtering MassIVE GNPS experiments into GeMS subsets (GeMS-A/B/C) and controlling redundancy via LSH clustering with per-cluster caps (e.g., 10 or 1,000 spectra). Initialize a 7-layer transformer with Graphormer-style attention over pairwise m/z Fourier-feature differences plus a precursor token, then pre-train it on GeMS with two self-supervised objectives: predicting masked spectral peaks and predicting chromatographic retention order between spectra pairs. The retention-order task injects chromatography-aware chemical information that, together with masked peak prediction, yields 1,024-dimensional real-valued embeddings for held-out spectra. Evaluate pre-training quality by projecting embeddings into a 1,023-dimensional PCA subspace, running k=3 linear clustering grouped by 2D InChI key (expect accuracy near 0.865), and progressively training linear probes on frozen representations to recover MACCS fingerprint bits (expect precision in the 0.81-0.84 regime with progressive fragment discovery during training).

## Related tools

- **pluskal-lab/DreaMS** (Reference implementation of the transformer architecture, self-supervised pre-training (masked m/z prediction + retention order prediction), and embedding extraction) — github.com__pluskal-lab__DreaMS
- **HDF5** (Compact binary storage format for GeMS spectra and LC-MS/MS metadata feeding the pre-training pipeline)

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- k=3 linear clustering accuracy on embeddings in the 1,023-D PCA subspace, grouped by 2D InChI key, approaches the reported value of 0.865
- Linear-probe precision for recovering MACCS fingerprint bits from frozen representations falls in the reported 0.81-0.84 regime, with precision improving progressively as pre-training depth increases
- Extracted embeddings are 1,024-dimensional real-valued vectors and, before fine-tuning, zero-shot cosine similarity in this space outperforms supervised MS2DeepScore on molecular similarity prediction
- Representation clusters remain organized by structural similarity across spectra acquired under different ionization and fragmentation settings

## Limitations

- The model was trained using only a subset of available mass spectra; training on the full repository-scale collection remains unexplored
- Spectra beyond positive ionization mode or singly charged precursor ions are not incorporated into training
- The method focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions

## Evidence

- [abstract] Retention-order task used for self-supervised pre-training: "pre-trained it on our GeMS data to predict masked spectral peaks and chromatographic retention orders"
- [abstract] Scale and supervision-free nature of the pre-training: "a large self-supervised neural network (with 116 million parameters) trained directly on the repository-scale collection of raw experimental mass spectra"
- [abstract] Quality of resulting representations: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [discussion] Data coverage caveat: "we trained our model using only a subset of available mass spectra"
- [methods] Environment setup for reproducing pre-training: "# Install requirements
pip install -r requirements.txt"
