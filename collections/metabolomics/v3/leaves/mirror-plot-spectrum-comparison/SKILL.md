---
name: mirror-plot-spectrum-comparison
description: Use when two USI-resolvable tandem MS spectra (e.g. GNPS/MassBank/MassIVE
  library or task spectra) must be compared side-by-side, with matched peaks highlighted
  and a cosine similarity score for the pair.
when_to_use_negative:
- Comparing more than two spectra — the /mirror/ endpoint supports only a pair of
  spectra (usi1 and usi2)
- The input is a single spectrum to visualize alone — use the /spectrum/ or /svg/
  single-spectrum endpoints instead of /mirror/
- You need proteomics peptide-spectrum match scoring rather than metabolomics spectral
  cosine comparison
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metabolomics Spectrum Resolver
  role: Resolves USIs, serves mirror plot renders via /mirror/, /svg/mirror, /png/mirror,
    and machine-readable peak lists via /json/ for the two spectra being compared
  repo: github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python library used for spectral preprocessing and rendering/plotting of the
    mirror comparison figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:dc78495c64b20fd06aac8829d7cfc7df1152783840c4a56f04128c7eeb207bf7
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mirror-plot-spectrum-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and visually compare two tandem MS spectra as a mirror plot using the Metabolomics Spectrum Resolver, including an optional shifted cosine similarity that also matches fragment peaks differing by the precursor mass difference between the spectra.

## When to use

Use when two USI-resolvable tandem MS spectra (e.g. GNPS/MassBank/MassIVE library or task spectra) must be compared side-by-side, with matched peaks highlighted and a cosine similarity score for the pair.

## When NOT to use

- Comparing more than two spectra — the /mirror/ endpoint supports only a pair of spectra (usi1 and usi2)
- The input is a single spectrum to visualize alone — use the /spectrum/ or /svg/ single-spectrum endpoints instead of /mirror/
- You need proteomics peptide-spectrum match scoring rather than metabolomics spectral cosine comparison

## Inputs

- Two Universal Spectrum Identifiers (USIs) resolving spectra from GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, or Metabolomics Workbench (e.g. mzspec:MASSBANK::accession:BSU00002)
- Spectrum peak lists (m/z and intensity arrays) fetched from the Spectrum Resolver /json/ endpoint

## Outputs

- Rendered mirror plot (SVG/PNG) with the two spectra plotted top and bottom and matching peaks highlighted
- CSV of similarity scores with columns cosine and shifted_cosine for the pair

## How to apply

Retrieve the peak lists (m/z and intensity arrays) for the two spectra from the Spectrum Resolver machine-readable /json/ endpoint using their USIs, which resolve spectra from GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, and Metabolomics Workbench. Optionally fetch the pre-rendered mirror plot via /svg/mirror or /png/mirror with usi1 and usi2 parameters. Preprocess each spectrum with spectrum_utils (restrict to fragment peaks below the precursor m/z and round m/z values to a fixed decimal precision) to form comparable peak vectors. Compute the standard cosine similarity as the intensity-weighted inner product over peaks matched within the fixed m/z tolerance, divided by the product of the spectral intensity norms. For the shifted cosine variant, additionally match peaks whose m/z values differ by the precursor mass difference between the two spectra and normalize the union of direct and shifted matches by the spectral norms. Write both scores to a CSV (columns cosine and shifted_cosine); matched peaks are highlighted in the rendered mirror plot so match quality can be assessed visually.

## Related tools

- **Metabolomics Spectrum Resolver** (Resolves USIs, serves mirror plot renders via /mirror/, /svg/mirror, /png/mirror, and machine-readable peak lists via /json/ for the two spectra being compared) — github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python library used for spectral preprocessing and rendering/plotting of the mirror comparison figures)

## Examples

```
from spectrum_utils import spectrum as sus; spec1, spec2 = sus.MsmsSpectrum.from_usi('mzspec:MASSBANK::accession:BSU00002'), sus.MsmsSpectrum.from_usi('mzspec:MASSBANK::accession:BSU00002'); cosine = sus.spectrum_similarity(spec1, spec2, shift=0)
```

## Evaluation signals

- Both similarity scores must lie in [0, 1]
- A self-comparison (identical spectrum against itself, e.g. BSU00002 as both usi1 and usi2) must yield a score of 1.0 for both cosine and shifted_cosine
- The output CSV exposes the cosine and shifted_cosine columns for the pair
- Rendered mirror plot shows matching peaks highlighted so visual match quality corresponds to the computed score
- The /mirror/ request succeeds for USIs from all supported repository types (GNPS task/library, MassBank accession, MassIVE, MetaboLights, Metabolomics Workbench)

## Limitations

- USI identifiers are based on draft USI and draft Metabolomics USI identifiers, specified as mzdraft instead of mzspec, and are subject to change
- The tool supports a pair of spectra per mirror plot, not arbitrary multi-spectrum alignment
- The tool is implemented as a web service, so local use requires either network access to the service endpoints or running the unit-tested codebase locally (pytest -v -n 4 ./test/)

## Evidence

- [intro] spectrum_utils plotting role: [evidence span withheld — anchor status: not_found]
