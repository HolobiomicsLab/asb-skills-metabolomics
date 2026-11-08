---
name: gray-code-precursor-mass-encoding
description: 'Use when reconstructing or reimplementing GLEAMS spectrum featurization:
  you need to build the exact 3010-feature input vector (Gray-coded precursor mass/m/z,
  one-hot charge, 2449 fragment bins, 500 reference-spectrum similarities) before
  applying the pretrained GLEAMS embedding network.'
when_to_use_negative:
- Input is already a 32-dimensional GLEAMS embedding or a GLEAMS_embed.npy array —
  featurization is only the pre-embedding step.
- The goal is peptide identification (e.g., an MSGF+ search) rather than spectrum
  featurization/clustering; GLEAMS featurization supports embedding and clustering,
  not de novo ID.
- Spectra are not HCD-style peptide MS/MS spectra with usable precursor mass/m/z and
  charge — the encoding is trained on 30 million MassIVE-KB (v1) PSMs from human HCD
  data.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: GLEAMS
  role: Implements the spectrum-to-3010-feature encoding and the neural network that
    maps it to the 32-dimensional embedding; configuration of the feature construction
    lives in gleams/config.py.
  repo: https://github.com/bittremieux/GLEAMS
- name: Python 3.8
  role: Runtime required to install and run GLEAMS' featurization and embedding pipeline
    on a CUDA-enabled GPU.
  repo: https://github.com/bittremieux/GLEAMS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:82969f2a19112c4021470ed5a5f05e1ee4eddd74b0a626545728d6430dda8626
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - Python
  - Python 3.8
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gray-code-precursor-mass-encoding

## Summary

Reproduce GLEAMS's input feature encoding: each MS/MS spectrum is converted to a 3010-dimensional feature vector combining Gray-coded precursor mass/m/z, one-hot charge, square-root-transformed binned fragment intensities, and dot-product similarities to 500 reference spectra before neural-network embedding.

## When to use

Use when reconstructing or reimplementing GLEAMS spectrum featurization: you need to build the exact 3010-feature input vector (Gray-coded precursor mass/m/z, one-hot charge, 2449 fragment bins, 500 reference-spectrum similarities) before applying the pretrained GLEAMS embedding network.

## When NOT to use

- Input is already a 32-dimensional GLEAMS embedding or a GLEAMS_embed.npy array — featurization is only the pre-embedding step.
- The goal is peptide identification (e.g., an MSGF+ search) rather than spectrum featurization/clustering; GLEAMS featurization supports embedding and clustering, not de novo ID.
- Spectra are not HCD-style peptide MS/MS spectra with usable precursor mass/m/z and charge — the encoding is trained on 30 million MassIVE-KB (v1) PSMs from human HCD data.

## Inputs

- MS/MS spectrum in a peak file (e.g., mzML) with precursor mass, precursor m/z, precursor charge, and peak list
- Invariant reference spectra set used for the 500 normalized dot-product similarity features

## Outputs

- 3010-dimensional feature vector per spectrum (27-bit Gray-coded precursor mass, 27-bit Gray-coded precursor m/z, one-hot charge, 2449 square-root-transformed fragment-intensity bins, 500 reference-spectrum dot-product similarities)
- 32-dimensional GLEAMS embedding per spectrum after applying the neural network (exported as GLEAMS_embed.npy)

## How to apply

Load each MS/MS spectrum (precursor mass, precursor m/z, precursor charge, and peak list) from a peak file such as mzML. Encode the precursor mass and precursor m/z each as a 27-bit Gray code and the precursor charge as a one-hot vector over 7 states. Bin fragment ions over the 50.5–2500 m/z range into 2449 bins and apply a square-root transform to the binned intensities, which dampens the dominance of high-intensity peaks. Compute normalized dot-product similarities between the spectrum and an invariant set of 500 reference spectra, then concatenate all feature blocks and verify the resulting vector is exactly 3010-dimensional. This featurized vector is what the pretrained network (weights gleams_82c0124b.hdf5) consumes to produce the 32-dimensional embedding used by 'gleams cluster'.

## Related tools

- **GLEAMS** (Implements the spectrum-to-3010-feature encoding and the neural network that maps it to the 32-dimensional embedding; configuration of the feature construction lives in gleams/config.py.) — https://github.com/bittremieux/GLEAMS
- **Python 3.8** (Runtime required to install and run GLEAMS' featurization and embedding pipeline on a CUDA-enabled GPU.) — https://github.com/bittremieux/GLEAMS

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Concatenated feature vector dimension is exactly 3010 for every spectrum.
- Fragment-intensity block length is 2449 bins spanning 50.5–2500 m/z, with square-root-transformed (not raw) intensities.
- Precursor mass and m/z blocks are 27-bit Gray codes; charge block is one-hot over 7 states.
- Similarity block contains 500 normalized dot-product values against the fixed reference spectra set.
- Feeding the encoded vector through the GLEAMS model (gleams_82c0124b.hdf5) yields a 32-dimensional embedding matching the n x 32 output of 'gleams embed'.

## Limitations

- Encoding details (27-bit Gray codes, 2449 bins, 500 references) are fixed by gleams/config.py; changing them requires retraining the network.
- The pretrained model weights must be present in the data/ directory; Git LFS bandwidth limits may block the default clone, requiring manual download from the v0.3 release.
- Requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.

## Evidence

- [methods] Encoding details: [evidence span withheld — anchor status: not_found]
- [readme] Embedding output: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [readme] Embed command: [evidence span withheld — anchor status: external_doc]
- [readme] Environment requirements: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
