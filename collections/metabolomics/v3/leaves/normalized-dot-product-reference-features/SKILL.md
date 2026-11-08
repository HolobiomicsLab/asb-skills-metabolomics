---
name: normalized-dot-product-reference-features
description: Use when encoding MS/MS spectra as neural-network input features and
  you need a fixed-size, comparable similarity block (500 dot products against reference
  spectra) alongside precursor Gray codes and binned fragment intensities.
when_to_use_negative:
- Input is already an embedding or feature table — this skill only applies to raw
  MS/MS peak lists being converted to GLEAMS input features
- You need the final 32-dimensional embeddings rather than the 3010-dimensional input
  encoding; use the `gleams embed` output directly instead
- The reference set is not fixed/invariant — recomputing similarities against a changing
  reference set breaks comparability across spectra
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GLEAMS
  role: Implements the spectrum encoding (precursor Gray codes, 2449 fragment intensity
    bins, 500 reference-spectrum dot products) into the 3010-dimensional input vector
  repo: https://github.com/bittremieux/GLEAMS
- name: ANN-SoLo
  role: Open modification search of GLEAMS cluster centroids that yielded the top
    500 precursor mass differences informing the 500-reference similarity block
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:757060a566dde1b4468f42978c8bdb5889ef509d156aea4432e8697f82c97174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Normalized dot-product reference features

## Summary

Computes normalized dot-product (cosine) similarities between each MS/MS spectrum and an invariant reference set of 500 spectra as one feature block of GLEAMS's 3010-dimensional spectrum encoding.

## When to use

Use when encoding MS/MS spectra as neural-network input features and you need a fixed-size, comparable similarity block (500 dot products against reference spectra) alongside precursor Gray codes and binned fragment intensities.

## When NOT to use

- Input is already an embedding or feature table — this skill only applies to raw MS/MS peak lists being converted to GLEAMS input features
- You need the final 32-dimensional embeddings rather than the 3010-dimensional input encoding; use the `gleams embed` output directly instead
- The reference set is not fixed/invariant — recomputing similarities against a changing reference set breaks comparability across spectra

## Inputs

- Single MS/MS spectrum (precursor mass, precursor m/z, charge, peak list) from a peak file (e.g. mzML)
- Fixed reference set of 500 spectra used for similarity computation

## Outputs

- 500-dimensional normalized dot-product similarity feature block
- 3010-dimensional concatenated GLEAMS input feature vector per spectrum

## How to apply

After encoding the precursor (27-bit Gray codes for mass and m/z, one-hot charge over 7 states) and binning fragment intensities square-root transformed into 2449 bins over 50.5–2500 m/z, compute the normalized dot product between the current spectrum and each spectrum in a fixed reference set of 500 spectra. The reference spectra are kept invariant so the similarity block is comparable across all spectra processed with the same model configuration; the reference-derived feature count (500) aligns with the top-500 precursor mass differences obtained from the ANN-SoLo open modification search of GLEAMS cluster centroids. Concatenate the similarity block with the precursor and fragment blocks and verify the total feature dimension is exactly 3010 before feeding the vector to the GLEAMS network that outputs a 32-dimensional embedding.

## Related tools

- **GLEAMS** (Implements the spectrum encoding (precursor Gray codes, 2449 fragment intensity bins, 500 reference-spectrum dot products) into the 3010-dimensional input vector) — https://github.com/bittremieux/GLEAMS
- **ANN-SoLo** (Open modification search of GLEAMS cluster centroids that yielded the top 500 precursor mass differences informing the 500-reference similarity block)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Concatenated feature vector dimension is exactly 3010 (27-bit Gray code blocks + 7-state charge one-hot + 2449 fragment bins + 500 similarity features)
- Fragment intensities are square-root transformed and binned over 50.5–2500 m/z into exactly 2449 bins
- Dot products are normalized, so each similarity feature falls in [0, 1]
- The reference set of 500 spectra remains invariant across all spectra encoded with the same model configuration

## Limitations

- The similarity block is only meaningful with the fixed 500-spectrum reference set; a different reference set changes the input distribution and would require retraining
- GLEAMS encoding is designed for HCD-style MS/MS spectra (trained on 30 million MassIVE-KB v1 PSMs); encodings from other fragmentation regimes may be out of distribution

## Evidence

- [methods] normalized dot products against an invariant 500-spectrum reference set: [evidence span withheld — anchor status: not_found]
- [results] 3010-dimensional feature vector with precursor, fragment, and similarity blocks: [evidence span withheld — anchor status: not_found]
- [supplementary] 500 references align with top-500 ANN-SoLo mass differences: "Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids."
- [readme] encoding feeds the neural network that produces 32-dim embeddings: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
