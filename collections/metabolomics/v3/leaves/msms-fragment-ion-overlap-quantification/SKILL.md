---
name: msms-fragment-ion-overlap-quantification
description: Use when LC–MS/MS spectra (.mgf) are embedded via an ms2deepscore similarity
  matrix and you need to characterize how much of the fragmentation signal is shared
  among spectra in a dense local neighborhood around a feature of interest.
when_to_use_negative:
- The dataset exceeds ~5000 features after preprocessing — the article reports the
  dashboard is only expected to scale to 5000 features and is untested beyond that
- The goal is fully automated annotation rather than manual fragmentation-overlap-driven
  structural hypothesis generation — the article frames overlap analysis as a starting
  point for manual annotation efforts
- ms2deepscore scores are needed but the underlying compound class is poorly covered
  by or unrelated to the ms2deepscore training data
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Interactive dashboard whose fragmap-style construction and threshold settings
    (default 0.7 minimum fragmentation overlap) define the overlap-quantification
    approach
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the .mgf MS/MS spectra and provides Spectrum objects to retrieve (m/z,
    intensity) peak tuples for binned fragment comparison
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Supplies the pairwise similarity matrix whose thresholded edges define the
    dense local neighborhood in which fragment overlaps are computed
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:da1953f276d41e8ae6f52dd9cfc0cf10b4dc8fd9565850eec1b265c9fab47977
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# msms-fragment-ion-overlap-quantification

## Summary

Quantifies pairwise MS/MS fragment-ion overlap within a densely connected similarity neighborhood (e.g., around a reference-standard feature such as Procyanidin-A2), exposing shared fragmentation patterns that, together with local topology and metadata, seed structural hypothesis generation and manual annotation.

## When to use

Use when LC–MS/MS spectra (.mgf) are embedded via an ms2deepscore similarity matrix and you need to characterize how much of the fragmentation signal is shared among spectra in a dense local neighborhood around a feature of interest.

## When NOT to use

- The dataset exceeds ~5000 features after preprocessing — the article reports the dashboard is only expected to scale to 5000 features and is untested beyond that
- The goal is fully automated annotation rather than manual fragmentation-overlap-driven structural hypothesis generation — the article frames overlap analysis as a starting point for manual annotation efforts
- ms2deepscore scores are needed but the underlying compound class is poorly covered by or unrelated to the ms2deepscore training data

## Inputs

- specXplore importing pipeline outputs: ms2deepscore pairwise similarity matrix and t-SNE embedding
- MS/MS spectra in .mgf format with the feature identifier key 'feature_id'
- A reference-standard or target feature node (e.g., the Procyanidin-A2 corresponding feature) localized on the t-SNE overview

## Outputs

- Fragment-ion overlap statistics table (e.g., procyanidin_a2_region_overlap_stats.csv) with region size, pairwise overlap fractions, and similarity-distribution summaries
- ms2deepscore similarity distribution summary for the isolated neighborhood, bounded in [0, 1]

## How to apply

Start from specXplore importing-pipeline outputs: the precomputed ms2deepscore pairwise similarity matrix (values bounded in [0, 1]) and the t-SNE embedding over the dataset (e.g., the urine metabolome set with <4000 features after preprocessing). Threshold the ms2deepscore matrix using the dashboard's default pairwise settings — edges requiring a minimum fragmentation overlap of 0.7 — to extract the densely connected neighborhood around the target feature node identified on the t-SNE overview. Retrieve the MS/MS peak tuples (m/z, intensity) for every spectrum in that neighborhood from the .mgf feature data via matchms, ensuring the feature identifier key is 'feature_id'. Compute pairwise fragment-ion overlap fractions among all spectra by comparing binned and sorted m/z fragments, in the manner of the specXplore fragmap construction, and compute summary statistics of the ms2deepscore similarity distribution within the region. Aggregate region size, overlap statistics, and similarity distribution summaries into a single results table (e.g., procyanidin_a2_region_overlap_stats.csv).

## Related tools

- **specXplore** (Interactive dashboard whose fragmap-style construction and threshold settings (default 0.7 minimum fragmentation overlap) define the overlap-quantification approach) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the .mgf MS/MS spectra and provides Spectrum objects to retrieve (m/z, intensity) peak tuples for binned fragment comparison) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Supplies the pairwise similarity matrix whose thresholded edges define the dense local neighborhood in which fragment overlaps are computed) — https://github.com/kevinmildau/specXplore

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The output table contains fragment-overlap and similarity-distribution fields with a nonzero feature count for the isolated region
- All ms2deepscore similarity values in the summary fall within [0, 1]
- Region edges were drawn only at the thresholded similarity level (default minimum fragmentation overlap 0.7)
- Overlap statistics reproduce the characteristic fragmentation overlap the paper reports for the Procyanidin-A2 feature group, alongside its higher experimental-spectra complexity
- Every spectrum contributing to overlap computation carries the 'feature_id' metadata key from the .mgf

## Limitations

- On macOS arm64 systems, the current ms2deepscore package may silently produce similarity predictions inconsistent with other systems (matchms/ms2deepscore issue 199) — no error or warning is raised, directly affecting importing-pipeline results
- specXplore currently works on macOS and Linux but fails on Windows, and the ms2deepscore backbone is only expected to work well for spectra/metabolites covered by or related to its training data
- Scalability beyond ~5000 features is untested; the dashboard is expected to face difficulties for larger datasets with liberal threshold settings

## Evidence

- [discussion] threshold default: [evidence span withheld — anchor status: not_found]
- [methods] matchms import: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [abstract] quantitative backbone: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone"
- [readme] README platform warning: [evidence span withheld — anchor status: external_doc]
- [readme] README arm64 caveat: [evidence span withheld — anchor status: external_doc]
