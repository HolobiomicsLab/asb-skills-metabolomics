---
name: msms-spectrum-preprocessing
description: Use when preparing raw or centroided query spectra and spectral library
  spectra (e.g. iPRG2012 TripleTOF data, NIST/Selevsek libraries) as input to ANN-SoLo
  or SpectraST-style library searching.
when_to_use_negative:
- Spectra have already been preprocessed with different, incompatible peak-count or
  intensity-normalization conventions — reapplying could double-filter or distort
  the remaining peaks
- The downstream tool expects raw intensities or its own internal preprocessing (e.g.
  MSFragger fragment-index database search, which is not spectral-library based)
- Spectra intentionally kept with few peaks or narrow mass ranges for targeted diagnostics,
  since the <10-peak / <250 Da discard rule will silently remove them
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Implements the preprocessing (precursor-peak removal, 1% noise filtering,
    50-peak cap, 10-peak/250-Da discard, rank transformation) on both query and library
    spectra before open search
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used upstream to concatenate libraries and remove duplicates before the library
    spectra are preprocessed for searching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:adf87d23fc1aeecb0ec8a40bca0fa3231f394d5ab01a5f3a3912d2bdd073f92b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9761904761904762
n_claims: 42
---

# msms-spectrum-preprocessing

## Summary

Preprocess query and spectral-library MS/MS spectra identically before open spectral library searching: remove the precursor peak, strip low-intensity noise, cap peak counts, discard poor spectra, and rank-transform intensities so similarity scores (dot product / shifted dot product) behave consistently.

## When to use

Use when preparing raw or centroided query spectra and spectral library spectra (e.g. iPRG2012 TripleTOF data, NIST/Selevsek libraries) as input to ANN-SoLo or SpectraST-style library searching.

## When NOT to use

- Spectra have already been preprocessed with different, incompatible peak-count or intensity-normalization conventions — reapplying could double-filter or distort the remaining peaks
- The downstream tool expects raw intensities or its own internal preprocessing (e.g. MSFragger fragment-index database search, which is not spectral-library based)
- Spectra intentionally kept with few peaks or narrow mass ranges for targeted diagnostics, since the <10-peak / <250 Da discard rule will silently remove them

## Inputs

- Query MS/MS spectra (e.g. iPRG2012 TripleTOF spectra from MassIVE MSV000078492)
- Spectral library spectra (concatenated NIST human HCD 2016/09/12 + Selevsek yeast library, with 1:1 target:decoy spectra)

## Outputs

- Preprocessed query spectra retained for cascade search (precursor peak removed, ≤50 peaks, ≥10 peaks, ≥250 Da mass range, rank-transformed intensities)
- Preprocessed library spectra used to build the ANN index
- Discarded-spectrum log / counts of spectra filtered by the minimum-peak and mass-range criteria

## How to apply

Apply the same preprocessing to both query and library spectra, since scores are only comparable when spectra are normalized the same way. First remove the precursor peak from each spectrum. Next remove noise peaks whose intensity is below 1% of the most intense peak, then restrict the spectrum to at most its 50 most intense peaks. Discard any spectrum with fewer than 10 remaining peaks or a mass range below 250 Da, as it carries too little fragment information for reliable matching. Finally rank-transform the peak intensities so the dot-product and shifted-dot-product scores depend on peak intensity ordering rather than raw magnitudes; with 0.02 Da fragment tolerance these cleaned spectra feed the cascade candidate-ranking stage.

## Related tools

- **ANN-SoLo** (Implements the preprocessing (precursor-peak removal, 1% noise filtering, 50-peak cap, 10-peak/250-Da discard, rank transformation) on both query and library spectra before open search) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used upstream to concatenate libraries and remove duplicates before the library spectra are preprocessed for searching)

## Evaluation signals

- Every retained spectrum has at most 50 peaks and no precursor peak remaining
- All spectra written for searching have ≥10 peaks and a mass range ≥250 Da; discarded spectra are excluded from the search set
- Peak intensities are rank-transformed (ordering-based), so the top-intensity peak has the same normalized value across spectra
- Identical preprocessing parameters are confirmed to have been applied to both query and library spectra (comparable SSM score distributions)
- Preprocessed inputs yield SSMs whose FDR-filtered identifications at 1% match the reported counts on iPRG2012

## Limitations

- Spectra with fewer than 10 peaks or a mass range below 250 Da are discarded entirely and cannot be identified
- Restricting to the 50 most intense peaks sacrifices low-abundance fragment peaks, which may affect sensitivity for some modified peptides
- Preprocessing must be applied symmetrically to query and library spectra; applying different filters breaks score comparability and FDR estimation

## Evidence

- [methods] Symmetric preprocessing requirement: "Prior to spectral library searching both the query spectra and library spectra are similarly preprocessed"
- [methods] 1% noise-peak filter: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] 50 most intense peaks cap: "the spectrum is further restricted to its 50 most intense peaks"
- [methods] Minimum-peak and mass-range discard rule: "any spectrum that contains fewer than 10 peaks remaining or with a mass range less than 250 Da is discarded"
- [readme] Search context in which preprocessing runs: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
