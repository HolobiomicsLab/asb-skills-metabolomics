---
name: fragment-intensity-binning-sqrt-transform
description: 'Use when reconstructing GLEAMS'' per-spectrum input encoding: fragment
  peaks from an mzML/peak file must be converted to fixed-length binned intensity
  features (2449 bins, 50.5–2500 m/z, square-root transformed) before feeding the
  neural network.'
when_to_use_negative:
- You already have the 32-dimensional GLEAMS embedding output — the neural network,
  not this raw feature encoding, is the relevant stage then (use `gleams embed`).
- You need spectral similarity for direct library matching rather than network input
  — GLEAMS embeddings or a tool like ANN-SoLo operate on the embedded/clustered representations
  instead.
- Training configuration changes (e.g. bin count or m/z range edited in gleams/config.py)
  would invalidate the fixed 2449-bin/50.5–2500 m/z scheme; do not assume these constants
  hold for custom configurations.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Provides the spectrum encoding pipeline that bins fragment intensities and
    assembles the 3010-dimensional input feature vector fed to the neural network
  repo: https://github.com/bittremieux/GLEAMS
- name: Python 3.8
  role: Runtime required to install and run GLEAMS encoding
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:1926eb8bfb39e7a22e090c7a923b4e37e74b6130f6d5ac6408d7cfd514c38ec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - Python
  - Python 3.8
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fragment-intensity-binning-sqrt-transform

## Summary

Encodes the fragment-ion content of a single MS/MS spectrum as part of GLEAMS' 3010-dimensional input feature vector by binning peak intensities into 2449 bins over the 50.5–2500 m/z range and applying a square-root transform, followed by concatenation with Gray-coded precursor features and 500 reference-spectrum similarities.

## When to use

Use when reconstructing GLEAMS' per-spectrum input encoding: fragment peaks from an mzML/peak file must be converted to fixed-length binned intensity features (2449 bins, 50.5–2500 m/z, square-root transformed) before feeding the neural network.

## When NOT to use

- You already have the 32-dimensional GLEAMS embedding output — the neural network, not this raw feature encoding, is the relevant stage then (use `gleams embed`).
- You need spectral similarity for direct library matching rather than network input — GLEAMS embeddings or a tool like ANN-SoLo operate on the embedded/clustered representations instead.
- Training configuration changes (e.g. bin count or m/z range edited in gleams/config.py) would invalidate the fixed 2449-bin/50.5–2500 m/z scheme; do not assume these constants hold for custom configurations.

## Inputs

- Single MS/MS spectrum from a peak file (e.g. mzML) containing precursor mass, precursor charge, and a fragment peak list (m/z, intensity)

## Outputs

- 2449-dimensional square-root-transformed binned fragment intensity vector (fragment block of the 3010-feature GLEAMS input)
- Concatenated 3010-dimensional per-spectrum input feature vector for the GLEAMS neural network

## How to apply

Load one MS/MS spectrum (precursor mass, charge, and peak list) from a peak file such as mzML. Bin the fragment ion intensities over the 50.5–2500 m/z range into 2449 bins, then apply a square-root transform to the binned intensities to damp the dominance of high-intensity peaks. Separately encode the precursor mass and m/z as 27-bit Gray codes and the precursor charge as a one-hot vector over 7 states, and compute normalized dot-product similarities between the spectrum and an invariant set of 500 reference spectra. Concatenate all feature blocks (Gray-coded precursor, 2449-bin fragment intensities, 500 similarities) and verify the resulting vector has exactly 3010 dimensions before passing it to the GLEAMS neural network.

## Related tools

- **GLEAMS** (Provides the spectrum encoding pipeline that bins fragment intensities and assembles the 3010-dimensional input feature vector fed to the neural network) — https://github.com/bittremieux/GLEAMS
- **Python 3.8** (Runtime required to install and run GLEAMS encoding)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Concatenated feature vector (Gray-code precursor block + 2449 fragment bins + 500 reference similarities) has dimension exactly 3010.
- Fragment intensity block is 2449 bins spanning 50.5–2500 m/z with no peaks binned outside that range.
- Binned intensities are square-root transformed, so the maximum value in the fragment block is bounded (sqrt of the original intensity scale).
- Precursor mass and m/z blocks are 27-bit Gray codes; charge block is one-hot over 7 states with exactly one active entry.
- Running the official `gleams embed` command on the same spectrum yields consistent features and a 32-dimensional embedding, confirming the upstream encoding is correct.

## Limitations

- The 2449-bin / 50.5–2500 m/z scheme is a fixed configuration; changing it requires editing `gleams/config.py` and would change the total feature dimensionality.
- The 500 reference-spectrum similarities depend on an invariant reference set; the encoding is only reproducible with that same reference set.
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU, so the encoding cannot be run outside that environment without adaptation.

## Evidence

- [supplementary] GLEAMS encodes each spectrum as a 3010-dimensional feature vector combining precursor attributes, binned fragment intensities, and dot-product similarities with reference spectra: [evidence span withheld — anchor status: not_found]
- [supplementary] fragments are square-root transformed and binned into 2449 bins (50.5-2500 m/z): [evidence span withheld — anchor status: not_found]
- [readme] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file.: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
