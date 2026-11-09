---
name: precursor-mass-difference-alignment
description: Use when comparing two tandem MS spectra (e.g. a library spectrum vs.
  a query spectrum of a related molecule) and you need to score match quality including
  peaks shifted by the precursor mass difference.
when_to_use_negative:
- Only a single spectrum is available and no pairwise comparison is requested
- The goal is peptide/proteomics spectral identification against ProteomeXchange data
  rather than metabolomics mirror-match quality assessment
- The comparison requires full peak-by-peak fragment assignment rather than an aggregate
  similarity score
edam_operation: null
tools:
- name: Metabolomics Spectrum Resolver
  role: Source of machine-readable peak data (/json/ endpoint) and mirror plot rendering
    of the matched spectrum pair
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python preprocessing of the retrieved peak lists (fragment m/z restriction,
    rounding to fixed precision) used to build the similarity computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:cece102f2365cef16592ed6573c149131f96041364e8c5aae7d678c3721c52c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  repo_url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - spectrum_utils
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# precursor-mass-difference-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes standard and shifted cosine similarity scores for a pair of MS/MS spectra, where the shifted variant additionally matches fragment peaks offset by the precursor mass difference between the spectra, as exposed by the Metabolomics Spectrum Resolver's mirror plots.

## When to use

Use when comparing two tandem MS spectra (e.g. a library spectrum vs. a query spectrum of a related molecule) and you need to score match quality including peaks shifted by the precursor mass difference.

## When NOT to use

- Only a single spectrum is available and no pairwise comparison is requested
- The goal is peptide/proteomics spectral identification against ProteomeXchange data rather than metabolomics mirror-match quality assessment
- The comparison requires full peak-by-peak fragment assignment rather than an aggregate similarity score

## Inputs

- Two Universal Spectrum Identifiers (USIs) resolvable by the Metabolomics Spectrum Resolver (e.g. mzspec:MASSBANK::accession:BSU00002, GNPS library accessions, MassIVE/MetaboLights/Metabolomics Workbench dataset scans)
- Peak lists (m/z and intensity arrays) retrieved from the /json/ or /proxi/v0.1/spectra endpoint
- Precursor m/z values for both spectra (to define the mass difference and fragment m/z ceiling)

## Outputs

- shifted_cosine_scores.csv with columns cosine and shifted_cosine for the spectrum pair
- Optional mirror plot rendering of the pair (SVG/PNG via /svg/mirror or /png/mirror) with matching peaks highlighted

## How to apply

Retrieve the m/z and intensity arrays for both spectra from the Metabolomics Spectrum Resolver machine-readable /json/ endpoint using their Universal Spectrum Identifiers (USIs). Preprocess each spectrum with spectrum_utils, restricting to fragment peaks below the precursor m/z and rounding m/z values to a fixed decimal precision to form comparable peak vectors. Compute the standard cosine similarity as the intensity-weighted inner product over peaks matched within a fixed m/z tolerance, divided by the product of the spectral intensity norms. For the shifted cosine, additionally match peaks whose m/z values differ by the precursor mass difference between the two spectra, forming the inner product over the union of direct and shifted matches normalized by the spectral norms. This reproduces the 'shifted cosine similarity' variant that the Spectrum Resolver supports alongside standard cosine in its mirror plots, where matching peaks are highlighted to assess match quality.

## Related tools

- **Metabolomics Spectrum Resolver** (Source of machine-readable peak data (/json/ endpoint) and mirror plot rendering of the matched spectrum pair) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python preprocessing of the retrieved peak lists (fragment m/z restriction, rounding to fixed precision) used to build the similarity computation)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/svg/mirror?usi1=mzspec:MASSBANK::accession:BSU00002&usi2=mzspec:MASSBANK::accession:BSU00002'
```

## Evaluation signals

- Both cosine and shifted_cosine scores must lie in [0, 1]
- A self-comparison (identical spectrum against itself) must yield a score of 1.0 for both variants
- shifted_cosine_scores.csv must expose exactly the cosine and shifted_cosine fields for the pair
- shifted_cosine should be >= plain cosine for a pair related by a precursor mass difference, since shifted matches are added to direct matches
- Peak vectors must exclude precursor peaks and use a fixed decimal m/z rounding so matches are deterministic

## Limitations

- USI identifiers are based on draft USI and draft Metabolomics USI specifications (mzdraft) and are subject to change
- The USI standard currently mainly supports proteomics spectral data via ProteomeXchange and only natively supports metabolomics repositories through the Spectrum Resolver's extensions
- Scores depend on the chosen fixed m/z tolerance / decimal rounding precision, which is not standardized across analyses

## Evidence

- [intro] plotting stack: [evidence span withheld — anchor status: not_found]
