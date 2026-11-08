---
name: spectral-embedding-pca-clustering
description: Use when you have held-out MS/MS spectra (e.g., GeMS HDF5) and want to
  verify that pre-trained spectral embeddings organize linearly by molecular structure,
  or to quantify structural clustering of spectra across differing ionization/fragmentation
  conditions.
when_to_use_negative:
- The goal is direct structure annotation or library retrieval from spectra — use
  fine-tuned DreaMS annotation heads or library search (e.g., against MoNA/NIST20/PubChem)
  instead of unsupervised clustering of pre-trained embeddings
- Spectra are outside the pre-training distribution described by the authors (e.g.,
  negative ionization mode, multiply charged precursor ions), where representation
  quality is not established
- The input is already a molecular structure table or fingerprint matrix — clustering
  spectra embeddings is redundant when structures, not spectra, are the objects of
  analysis
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: pluskal-lab/DreaMS
  role: Provides the pre-trained transformer that produces the 1,024-dimensional spectral
    embeddings, plus GeMS HDF5 loading utilities
  repo: https://github.com/pluskal-lab/DreaMS
- name: HDF5
  role: Binary storage format for the GeMS spectra and LC–MS/MS metadata consumed
    during embedding extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:827c6d03a12522ec2404fd1caaba6479324a4b3f9f95196fa3ea960bb5932bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - HDF5
  - pluskal-lab/DreaMS
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-embedding-pca-clustering

## Summary

Extracts 1,024-dimensional real-valued DreaMS embeddings from tandem MS/MS spectra and tests whether the representation space linearly clusters spectra by molecular structure (via PCA to 1,023 dims and k=3 clustering on 2D InChI-key groups), the core validation that self-supervised pre-training learns structure-aware representations.

## When to use

Use when you have held-out MS/MS spectra (e.g., GeMS HDF5) and want to verify that pre-trained spectral embeddings organize linearly by molecular structure, or to quantify structural clustering of spectra across differing ionization/fragmentation conditions.

## When NOT to use

- The goal is direct structure annotation or library retrieval from spectra — use fine-tuned DreaMS annotation heads or library search (e.g., against MoNA/NIST20/PubChem) instead of unsupervised clustering of pre-trained embeddings
- Spectra are outside the pre-training distribution described by the authors (e.g., negative ionization mode, multiply charged precursor ions), where representation quality is not established
- The input is already a molecular structure table or fingerprint matrix — clustering spectra embeddings is redundant when structures, not spectra, are the objects of analysis

## Inputs

- Held-out MS/MS spectra in GeMS HDF5 format (compact binary format with selected LC–MS/MS metadata, from MassIVE GNPS)
- Pre-trained DreaMS transformer model (116M parameters; weights e.g. Zenodo 10997887)
- Per-spectrum molecular identifiers (2D InChI keys) for grouping
- MACCS molecular fingerprint bits for linear probing

## Outputs

- 1,024-dimensional real-valued DreaMS embedding per spectrum
- 1,023-dimensional PCA-projected embedding matrix
- k=3 clustering accuracy grouped by 2D InChI key (expected ~0.865)
- Per-probing-depth precision of MACCS fingerprint bit recovery (expected 0.81–0.84)

## How to apply

Load held-out spectra from the GeMS-A10 HDF5 subset (mined from MassIVE GNPS) and pass them through the pre-trained 7-layer transformer (Graphormer-style attention over pairwise m/z Fourier-feature differences, with a precursor token) to obtain 1,024-dimensional real-valued embeddings. Group spectra by 2D InChI key, project the embeddings into a 1,023-dimensional PCA subspace, and run k=3 linear clustering within groups, scoring clustering accuracy against the reported ~0.865. As a complementary depth probe, progressively train linear probes on the frozen representations to recover MACCS molecular fingerprint bits and check per-depth precision falls in the reported 0.81–0.84 regime. Rationale: the paper shows these representations are 'organized according to the structural similarity between molecules' and remain robust across different MS conditions, so structural clustering accuracy is a direct readout of representation quality.

## Related tools

- **pluskal-lab/DreaMS** (Provides the pre-trained transformer that produces the 1,024-dimensional spectral embeddings, plus GeMS HDF5 loading utilities) — https://github.com/pluskal-lab/DreaMS
- **HDF5** (Binary storage format for the GeMS spectra and LC–MS/MS metadata consumed during embedding extraction)

## Examples

```
pip install -r requirements.txt && ln -s ../tutorials tutorials
```

## Evaluation signals

- k=3 clustering accuracy in the 1,023-dimensional PCA subspace (grouped by 2D InChI key) approaches the reported 0.865
- Linear-probing precision for MACCS fingerprint bits across training depths falls in the reported 0.81–0.84 regime
- Every embedding is a 1,024-dimensional real-valued vector (schema check on the embedding matrix)
- Clustering accuracy holds across spectra acquired under different ionization and fragmentation settings (robustness check)
- Zero-shot cosine similarity between embeddings outperforms the MS2DeepScore supervised baseline on molecular similarity prediction

## Limitations

- The reported linear clustering behavior was demonstrated on positive-mode, singly charged spectra; representations for other MS regimes are untested
- Pre-training used only a subset of available mass spectra, so clustering performance on under-represented structural classes may be weaker
- The method relies on tandem MS spectra alone, disregarding MS1 isotopic patterns and adduct distributions, which limits structural resolution for isobaric/isomeric compounds

## Evidence

- [abstract] Organized representations: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [abstract] Zero-shot similarity baseline: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] HDF5 input format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [readme] DreaMS repository: "DreaMS (Deep Representations Empowering the Annotation of Mass Spectra)"
