---
name: masked-peak-pretraining-transformer
description: Use when you have a large collection of raw unannotated MS/MS spectra
  (e.g., GeMS HDF5 mined from MassIVE GNPS) and want structure-aware embeddings without
  labeled spectral libraries.
when_to_use_negative:
- Your spectra are already annotated with library structures and you need direct annotation
  rather than transferable representations
- Your data are outside the model's training domain (negative ionization mode or multiply
  charged precursors), which the pre-training corpus did not cover
- You need MS1 isotopic pattern or adduct information exploited — the method is focused
  solely on tandem mass spectra
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: DreaMS
  role: Transformer architecture and self-supervised pre-training code (masked peak
    prediction, retention order prediction) over GeMS spectra
  repo: github.com__pluskal-lab__DreaMS
- name: HDF5
  role: Compact binary storage format for GeMS spectra and LC–MS/MS metadata feeding
    the pre-training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:4a575fbfe1767e69836b7ec556aea67d870dd8f3ede1ef9efc676b3d88de4bc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DreaMS
  - HDF5
  - pluskal-lab/DreaMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# masked-peak-pretraining-transformer

## Summary

Self-supervised pre-training of a transformer (DreaMS) on raw unannotated tandem mass spectra, using masked m/z-peak prediction and chromatographic retention order prediction as pretext tasks on the GeMS corpus, to yield 1,024-dimensional molecular representations organized by structural similarity.

## When to use

Use when you have a large collection of raw unannotated MS/MS spectra (e.g., GeMS HDF5 mined from MassIVE GNPS) and want structure-aware embeddings without labeled spectral libraries.

## When NOT to use

- Your spectra are already annotated with library structures and you need direct annotation rather than transferable representations
- Your data are outside the model's training domain (negative ionization mode or multiply charged precursors), which the pre-training corpus did not cover
- You need MS1 isotopic pattern or adduct information exploited — the method is focused solely on tandem mass spectra

## Inputs

- GeMS spectra in HDF5-based binary format with LC–MS/MS metadata (e.g., GeMS-A10 subset from MassIVE GNPS)
- Retention-time/order metadata for pairs of spectra from the same LC run (for the retention order objective)

## Outputs

- Pre-trained 116M-parameter DreaMS transformer checkpoint
- 1,024-dimensional real-valued DreaMS embeddings per input MS/MS spectrum

## How to apply

Load the GeMS spectra (e.g., the GeMS-A10 subset) stored in the compact HDF5-based binary format designed for deep learning, which includes selected LC–MS/MS metadata. Initialize the DreaMS transformer (116M parameters; 7 layers with Graphormer-style attention over pairwise m/z Fourier-feature differences plus a precursor token) and pre-train it self-supervised with two pretext objectives: prediction of masked spectral peaks and prediction of chromatographic retention order between spectra pairs. Address redundancy in the training corpus by LSH clustering with per-cluster random sampling caps before training. After pre-training, extract 1,024-dimensional real-valued embeddings for held-out spectra and verify quality via PCA projection (1,023-D subspace) with k=3 linear clustering grouped by 2D InChI key (expect ~0.865 accuracy) and progressive linear probing for MACCS fingerprint bits (expect 0.81–0.84 precision across probing depths).

## Related tools

- **DreaMS** (Transformer architecture and self-supervised pre-training code (masked peak prediction, retention order prediction) over GeMS spectra) — github.com__pluskal-lab__DreaMS
- **HDF5** (Compact binary storage format for GeMS spectra and LC–MS/MS metadata feeding the pre-training)

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- k=3 linear clustering accuracy of embeddings in the 1,023-D PCA subspace (grouped by 2D InChI key) approaches the reported ~0.865
- Linear-probing precision for recovering MACCS fingerprint bits from frozen representations falls in the reported 0.81–0.84 regime and increases progressively with probing depth
- Cosine similarity in the pre-trained (zero-shot) embedding space outperforms the supervised MS2DeepScore baseline on molecular similarity prediction
- Embedding structure is robust across spectra acquired under different ionization and fragmentation settings

## Limitations

- Training used only a subset of available mass spectra; the full potential of training on raw unannotated spectra remains to be unlocked
- Pre-training data exclude negative ionization mode and multiply charged precursor ions
- The method disregards MS1 isotopic patterns and adduct distributions, focusing solely on tandem mass spectra

## Evidence

- [abstract] Pre-training objectives on GeMS: "pre-trained it on our GeMS data to predict masked spectral peaks and chromatographic retention orders"
- [abstract] Model scale and training data: "a large self-supervised neural network (with 116 million parameters) trained directly on the repository-scale collection of raw experimental mass spectra"
- [abstract] HDF5 storage of GeMS: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Representation properties: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [methods] Environment setup from repo: "# Install requirements
pip install -r requirements.txt"
