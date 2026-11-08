---
name: mass-spectrum-preprocessing
description: Use when preparing MGF/mzML spectra for spectral library search with
  ANN-SoLo or dot-product-based matching — both library and query spectra must share
  the same peak filtering and normalization before candidate selection or scoring.
when_to_use_negative:
- When spectra are raw profiles that first need centroiding/peak picking at the instrument-conversion
  stage — this skill assumes peak-list input (e.g. msconvert output)
- When performing a sequence database search (e.g. MSFragger) where theoretical fragments
  rather than library peak lists are compared — the peak-count and intensity-rank
  rules are specific to spectral library scoring
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Implements the preprocessing step for both query and library spectra prior
    to candidate selection and SSM scoring
  repo: https://github.com/bittremieux/ANN-SoLo
- name: msconvert
  role: Converts raw instrument files to peak-list format before preprocessing (e.g.
    HEK293 raw files to MGF)
- name: SpectraST
  role: Used to concatenate library files and remove duplicates before preprocessing
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:11473d6a49833eba3235aceacfee6982627ec5db1716a1b3f8f3d9b034a34f20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectrum-preprocessing

## Summary

Preprocess query and library mass spectra identically before ANN-SoLo open spectral library searching: denoise, cap peak count, discard sparse spectra, and rank-transform intensities so that dot-product similarity and ANN indexing operate on a uniform, unit-normalized representation.

## When to use

Use when preparing MGF/mzML spectra for spectral library search with ANN-SoLo or dot-product-based matching — both library and query spectra must share the same peak filtering and normalization before candidate selection or scoring.

## When NOT to use

- When spectra are raw profiles that first need centroiding/peak picking at the instrument-conversion stage — this skill assumes peak-list input (e.g. msconvert output)
- When performing a sequence database search (e.g. MSFragger) where theoretical fragments rather than library peak lists are compared — the peak-count and intensity-rank rules are specific to spectral library scoring

## Inputs

- Query spectra as peak lists (MGF or mzML converted from raw files, e.g. HEK293 or iPRG2012 data)
- Spectral library spectra (e.g. NIST human HCD library version 2016/09/12, MassIVE-KB version 2017/11/27, or TripleTOF yeast library)

## Outputs

- Preprocessed library spectra ready for ANN indexing (unit-normalized vectors with rank-transformed intensities)
- Preprocessed query spectra for candidate retrieval and dot-product/shifted-dot-product scoring
- Discarded spectra filtered out due to insufficient peaks (<10) or narrow mass range (<250 Da)

## How to apply

Preprocess query spectra and library spectra identically. Remove precursor peaks and noise peaks below 1% of the intensity of the most intense peak, then restrict each spectrum to its 50 most intense peaks. Discard any spectrum containing fewer than 10 remaining peaks or with a mass range less than 250 Da. Convert the spectra to vector representations (1 Da mass bins for the ANN index, with per-bin intensities summed and vectors normalized to unit length) and rank-transform peak intensities so the dot product reflects peak rank rather than absolute intensity. Apply the same steps to both search levels of the cascade (level 1 dot product, level 2 shifted dot product) — inconsistent preprocessing between library and query spectra invalidates the similarity scores and downstream FDR estimation.

## Related tools

- **ANN-SoLo** (Implements the preprocessing step for both query and library spectra prior to candidate selection and SSM scoring) — https://github.com/bittremieux/ANN-SoLo
- **msconvert** (Converts raw instrument files to peak-list format before preprocessing (e.g. HEK293 raw files to MGF))
- **SpectraST** (Used to concatenate library files and remove duplicates before preprocessing)

## Examples

```
ann_solo search library.spectra.pkl query.mgf --precursor_tolerance 0.02 --fragment_tolerance 0.02 --peak_filter 0.01 --max_peaks 50
```

## Evaluation signals

- Both library and query spectra have undergone identical noise filtering: no peaks below 1% of base-peak intensity remain and at most 50 peaks per spectrum are retained
- No retained spectrum has fewer than 10 peaks or a mass range below 250 Da; discarded spectra count is logged
- Vectors used for ANN indexing are normalized to unit length with intensities rank-transformed
- Preprocessed library retains the 1:1 target-decoy ratio (shuffle-and-reposition decoys) so FDR estimation is valid after filtering
- Search behavior matches a control: the same spectrum preprocessed twice yields identical peak sets, and level-1/level-2 cascade results are consistent with deposited identification counts (e.g. PXD009861)

## Limitations

- Removing peaks below 1% of base-peak intensity and capping at 50 peaks discards weak fragment ions, which may reduce sensitivity for low-intensity diagnostic peaks
- Spectra with fewer than 10 peaks or <250 Da mass range are lost entirely; extremely short or sparse spectra cannot be searched
- FDR may be underestimated when small subgroups are used for open-search FDR filtering; preprocessing errors propagate into incorrect charge-normalized precursor mass-difference subgrouping
- The exact set of preprocessing cutoffs (1%, 50 peaks, 10 peaks, 250 Da) was tuned for high-resolution HCD/TOF data in this study and may not transfer directly to low-resolution or different instrument types

## Evidence

- [methods] Preprocessing applies to both query and library spectra: "Prior to spectral library searching both the query spectra and library spectra are similarly preprocessed"
- [methods] Noise-peak removal cutoff: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] Peak-count restriction and sparse-spectrum discarding: [evidence span withheld — anchor status: not_found]
- [readme] README framing of preprocessing role in the search engine: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
