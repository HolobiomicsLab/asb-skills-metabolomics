---
name: spectral-peak-matching
description: Use when two tandem MS spectra (e.g. a mirror-plot pair identified by
  USIs) must be compared numerically to score how many fragment peaks match, including
  peaks shifted by the precursor mass difference between the spectra.
when_to_use_negative:
- You only need a visual comparison — the Spectrum Resolver /svg/mirror or /png/mirror
  endpoint already renders mirror plots with highlighted matching peaks
- Input is raw profile-mode MS1 data rather than peak-picked MS/MS fragment spectra
- The spectra are not addressable via USI or the supported repositories (GNPS, MassBank,
  MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics Spectrum Resolver
  role: Source of machine-readable peak lists for the two spectra via the /json/ endpoint
    and reference implementation of mirror-plot peak matching
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: 'Python preprocessing of retrieved spectra: fragment m/z restriction below
    precursor m/z, m/z rounding to fixed precision, and cosine/shifted-cosine similarity
    computation'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:071d4fad3e7f86f8d18654842ab3e4f77350763fe42c60c32cd082c9c389ed4c
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-peak-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes intensity-weighted peak-matching similarity between two MS/MS spectra — standard cosine and a shifted cosine variant that additionally matches peaks differing by the precursor mass difference — using peaks retrieved from the Metabolomics Spectrum Resolver and processed with spectrum_utils.

## When to use

Use when two tandem MS spectra (e.g. a mirror-plot pair identified by USIs) must be compared numerically to score how many fragment peaks match, including peaks shifted by the precursor mass difference between the spectra.

## When NOT to use

- You only need a visual comparison — the Spectrum Resolver /svg/mirror or /png/mirror endpoint already renders mirror plots with highlighted matching peaks
- Input is raw profile-mode MS1 data rather than peak-picked MS/MS fragment spectra
- The spectra are not addressable via USI or the supported repositories (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange)

## Inputs

- Two Universal Spectrum Identifiers (USIs) resolvable by the Metabolomics Spectrum Resolver (e.g. mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077 or mzspec:MASSBANK::accession:BSU00002)
- Peak lists (m/z and intensity arrays) retrieved via the /json/ or /proxi/v0.1/spectra endpoints
- Precursor m/z values for each spectrum (to bound fragment peaks and derive the mass difference)

## Outputs

- Cosine similarity score in [0, 1] for the spectrum pair
- Shifted cosine similarity score in [0, 1] including precursor-mass-difference-shifted matches
- CSV (e.g. shifted_cosine_scores.csv) with columns cosine and shifted_cosine

## How to apply

Retrieve the peak lists (m/z and intensity arrays) for both spectra from the Metabolomics Spectrum Resolver machine-readable /json/ endpoint using their USIs, then preprocess each with spectrum_utils: restrict to fragment peaks below the precursor m/z and round m/z values to a fixed decimal precision to form comparable peak vectors. Compute the standard cosine similarity as the intensity-weighted inner product over peaks matched within the fixed m/z tolerance, divided by the product of the spectral intensity norms. For the shifted cosine variant, additionally match peaks whose m/z values differ by the precursor mass difference between the two spectra, and form the inner product over the union of direct and shifted matches normalized by the spectral norms. This mirrors what the Spectrum Resolver's /mirror/ endpoint renders, where matching peaks are highlighted to assess match quality.

## Related tools

- **Metabolomics Spectrum Resolver** (Source of machine-readable peak lists for the two spectra via the /json/ endpoint and reference implementation of mirror-plot peak matching) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python preprocessing of retrieved spectra: fragment m/z restriction below precursor m/z, m/z rounding to fixed precision, and cosine/shifted-cosine similarity computation)

## Examples

```
import requests; peaks=[requests.get(f'https://metabolomics-usi.ucsd.edu/json/?usi={u}').json() for u in (usi1, usi2)]
```

## Evaluation signals

- Both similarity scores must lie in [0, 1]
- A self-comparison (identical spectrum against itself, e.g. usi1 == usi2 as in the README mirror example) must yield 1.0 for both cosine and shifted_cosine
- Output CSV must expose exactly the cosine and shifted_cosine fields for the pair
- Retrieved peak arrays from /json/ must be non-empty and fragment peaks must all fall below each spectrum's precursor m/z
- Shifted matches should only pair peaks whose m/z difference equals the precursor mass difference between the two spectra

## Limitations

- Metabolomics USIs were draft identifiers at the time (prefixed mzdraft instead of mzspec) and 'subject to change'
- The USI standard originally mainly supported proteomics data via ProteomeXchange and needed a metabolomics extension for GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench
- Matching quality depends on the fixed m/z decimal-precision rounding used to bin peaks; no changelog or software version is stated for the Resolver

## Evidence

- [full_text] The Spectrum Resolver's mirror plots support a 'shifted cosine similarity' variant alongside the standard cosine similarity, in which peaks that differ by the precursor mass difference between the two spectra are also matched, with matching peaks highlighted to assess match quality.: [evidence span withheld — anchor status: not_found]
- [intro] Plotting and spectral processing implemented with spectrum_utils in Python.: [evidence span withheld — anchor status: not_found]
