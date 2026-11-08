---
name: mirror-plot-cosine-similarity-spectral-matching
description: Use when an experimental MS/MS spectrum must be compared against a spectral
  library match (or another dataset spectrum) and the deliverable is a publication-ready
  mirror plot with a quantified cosine similarity and highlighted matching peaks.
when_to_use_negative:
- Only a single spectrum needs to be visualized — use the plain /png/ or /svg/ rendering
  endpoints instead of /mirror
- A machine-readable peak list or metadata export is needed rather than a figure —
  use the /json/, /csv/, or /proxi/v0.1/spectra endpoints
- The spectra cannot be addressed by a supported USI construct (the resolver supports
  specific draft-USI formats for GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and
  Metabolomics Workbench sources)
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that retrieves both USI-addressed spectra and renders the mirror
    plot with cosine similarity via /svg/mirror and /png/mirror endpoints
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library used by the resolver's plotting functionality; provides the
    dot-product spectral matching for the cosine similarity score
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics
    use cases, used to address both input spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:ef3fc7d4f85c76994d43ab082b72127af843c998fbcd1df8595a6555aebe8319
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
  - USI
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mirror-plot-cosine-similarity-spectral-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render two tandem MS spectra as a mirror plot via the Metabolomics Spectrum Resolver's /mirror endpoints by supplying their USIs, quantifying the match with standard cosine similarity and highlighting shared fragments between the top and bottom spectra.

## When to use

Use when an experimental MS/MS spectrum must be compared against a spectral library match (or another dataset spectrum) and the deliverable is a publication-ready mirror plot with a quantified cosine similarity and highlighted matching peaks.

## When NOT to use

- Only a single spectrum needs to be visualized — use the plain /png/ or /svg/ rendering endpoints instead of /mirror
- A machine-readable peak list or metadata export is needed rather than a figure — use the /json/, /csv/, or /proxi/v0.1/spectra endpoints
- The spectra cannot be addressed by a supported USI construct (the resolver supports specific draft-USI formats for GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench sources)

## Inputs

- Two Universal Spectrum Identifiers (USIs) addressing spectra from supported sources (GNPS spectral libraries, MassBank, MassIVE, MetaboLights, Metabolomics Workbench, MS2LDA/MOTIFDB, ProteomeXchange)
- Optional plotting parameters: mz_min, mz_max, max_intensity, annotate_peaks (two nested lists for mirror plot), plot_title

## Outputs

- Mirror plot image (SVG or PNG) with both spectra, cosine similarity score, and matching peaks highlighted
- Rendered figure file (e.g. mirror_match.png)

## How to apply

Construct USIs for both spectra following the metabolomics-extended conventions, e.g. `mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077` for a GNPS library entry and `mzspec:MSV000082791:(-)-epigallocatechin:scan:2` for a MetaboLights dataset spectrum. Issue an HTTP request to the Spectrum Resolver's /svg/mirror or /png/mirror endpoint with usi1 and usi2 query parameters so both spectra are retrieved across resources (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench) and rendered as a mirror plot. The resolver computes the standard cosine similarity (spectrum_utils dot-product spectral matching) between the two peak lists and pairs shared fragments visually between the top and bottom spectra. Optionally tune the rendering with parameters such as mz_min, mz_max, max_intensity (e.g. max_intensity=150 for intensity scaling), and annotate_peaks given as two nested lists for the top and bottom spectrum. Save the retrieved figure (e.g. mirror_match.png); the underlying plotting is implemented in Python with spectrum_utils, Matplotlib, and Seaborn.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that retrieves both USI-addressed spectra and renders the mirror plot with cosine similarity via /svg/mirror and /png/mirror endpoints) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library used by the resolver's plotting functionality; provides the dot-product spectral matching for the cosine similarity score)
- **USI** (HUPO-PSI Universal Spectrum Identifier standard, extended for metabolomics use cases, used to address both input spectra)

## Examples

```
curl -o mirror_match.png 'https://metabolomics-usi.ucsd.edu/png/mirror?usi1=mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077&usi2=mzspec:MSV000082791:(-)-epigallocatechin:scan:2'
```

## Evaluation signals

- The figure file (e.g. mirror_match.png) exists and contains both spectra rendered as a mirror plot
- The rendered image displays the cosine similarity score between the two peak lists
- Matching peaks are highlighted and visually paired between the top and bottom spectra
- The HTTP request to /svg/mirror or /png/mirror with usi1 and usi2 returns a valid image rather than an error
- Expert review confirms the mirror plot correctly represents the experimental spectrum against its spectral library match

## Limitations

- USIs use the draft metabolomics USI specification — identifiers were specified as `mzdraft` instead of `mzspec` at some stages and are subject to change
- The native USI standard mainly supports proteomics spectral data from ProteomeXchange; metabolomics repositories are only covered via the metabolomics extension
- Generating SPLASH identifiers as an alternative is non-trivial because they involve a complex hashing operation

## Evidence

- [methods] Mirror plot reports cosine similarity between spectra: [evidence span withheld — anchor status: not_found]
- [intro] Plotting implemented with spectrum_utils: [evidence span withheld — anchor status: not_found]
