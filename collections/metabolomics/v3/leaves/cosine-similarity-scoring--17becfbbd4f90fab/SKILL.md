---
name: cosine-similarity-scoring
description: Use when a mirror plot comparison of two tandem MS spectra is needed
  and peaks that differ by the precursor mass difference between the two spectra must
  also be matched, as the Spectrum Resolver does for metabolomics USI pairs.
when_to_use_negative:
- When only a single spectrum must be rendered/annotated rather than compared to a
  second spectrum — the Spectrum Resolver's single-spectrum /svg/ and /png/ endpoints
  do not involve similarity scoring
- When the input is a set of library spectra to be clustered by spectral networking
  rather than a specific two-spectrum mirror match comparison
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Provides the machine-readable /json/ endpoint from which the peak lists (m/z
    and intensities) of the two spectra are retrieved via their USIs, and whose mirror
    plots display the resulting similarity scores
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library used to preprocess the retrieved spectra (restrict to fragment
    peaks below the precursor m/z and round m/z to fixed decimal precision) before
    computing the cosine and shifted cosine inner products
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:448e6542b1199b2c6391fa5b0db825d09f0e9fec4f83ec9feb2ad527aab4c759
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cosine-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes spectral similarity between two MS/MS spectra using cosine and shifted cosine scoring, where the shifted variant additionally matches peaks that differ by the precursor mass difference between the spectra, as used for mirror-plot match assessment in the Metabolomics Spectrum Resolver.

## When to use

Use when a mirror plot comparison of two tandem MS spectra is needed and peaks that differ by the precursor mass difference between the two spectra must also be matched, as the Spectrum Resolver does for metabolomics USI pairs.

## When NOT to use

- When only a single spectrum must be rendered/annotated rather than compared to a second spectrum — the Spectrum Resolver's single-spectrum /svg/ and /png/ endpoints do not involve similarity scoring
- When the input is a set of library spectra to be clustered by spectral networking rather than a specific two-spectrum mirror match comparison

## Inputs

- Peak lists (m/z and intensity arrays) for two MS/MS spectra, retrievable via the Metabolomics Spectrum Resolver /json/ endpoint given their USIs
- Precursor m/z values for both spectra (to derive the precursor mass difference used in the shifted cosine variant)

## Outputs

- cosine similarity score in [0, 1]
- shifted cosine similarity score in [0, 1]
- shifted_cosine_scores.csv with columns cosine and shifted_cosine for the spectrum pair

## How to apply

Retrieve the peak lists (m/z and intensity arrays) for the two spectra from the Metabolomics Spectrum Resolver machine-readable /json/ endpoint using their Universal Spectrum Identifiers (USIs). Preprocess each spectrum with spectrum_utils, restricting to fragment peaks below the precursor m/z and rounding m/z values to a fixed decimal precision to form comparable peak vectors. Compute the standard cosine similarity as the intensity-weighted inner product over peaks matched within the fixed m/z tolerance, divided by the product of the spectral intensity norms. For the shifted cosine variant, additionally match peaks whose m/z values differ by the precursor mass difference between the two spectra, then form the intensity inner product over the union of direct and shifted matches normalized by the spectral norms. Write both scores to a table (e.g. shifted_cosine_scores.csv with columns 'cosine' and 'shifted_cosine') so the pair's match quality can be compared with and without the precursor mass shift allowance.

## Related tools

- **Metabolomics Spectrum Resolver** (Provides the machine-readable /json/ endpoint from which the peak lists (m/z and intensities) of the two spectra are retrieved via their USIs, and whose mirror plots display the resulting similarity scores) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library used to preprocess the retrieved spectra (restrict to fragment peaks below the precursor m/z and round m/z to fixed decimal precision) before computing the cosine and shifted cosine inner products) — github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077' | python score_shifted_cosine.py --usi2 mzspec:MASSBANK::accession:SM858102 --out shifted_cosine_scores.csv
```

## Evaluation signals

- Both similarity scores must lie in [0, 1]
- A self-comparison (an identical spectrum against itself) must yield a score of 1.0 for both the cosine and shifted cosine variants
- The output CSV must expose the cosine and shifted_cosine fields for the spectrum pair
- Matching peaks in the shifted variant include those differing by the precursor mass difference, not only directly aligned peaks
- Plotting parameters (e.g. m/z rounding precision, fragment peak cutoff below precursor m/z) must be fixed identically for both spectra before scoring

## Limitations

- The shifted cosine variant depends on accurate precursor m/z values for both spectra; an incorrect precursor mass difference will mis-match shifted peaks
- The m/z rounding precision is a fixed parameter that determines the tolerance for direct peak matching and must be applied consistently
- USIs used by the Spectrum Resolver are based on draft USI and draft metabolomics USI identifiers, subject to change, and are specified as `mzdraft` instead of `mzspec` in the first block

## Evidence

- [results] shifted cosine similarity variant: [evidence span withheld — anchor status: not_found]
- [intro] plotting functionality implemented with spectrum_utils: [evidence span withheld — anchor status: not_found]
- [abstract] machine-readable export: "allow for consistent data export (in human and machine-readable forms) and publication-ready visualisations of tandem mass spectrometry spectra"
- [readme] draft USI identifiers caveat: "These identifiers are based on draft USI and draft Metabolomics USI identifiers. Thus, they are subject to change"
- [abstract] USI specification support: "This tool supports the Human Proteome Organization - Proteomics Standards Initiatives Universal Spectrum Identifier (USI) specification, which has been extended to deal with the metabolomics use cases"
