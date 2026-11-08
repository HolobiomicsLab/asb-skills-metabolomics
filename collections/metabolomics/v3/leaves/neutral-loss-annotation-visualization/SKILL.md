---
name: neutral-loss-annotation-visualization
description: Use when MS/MS spectra of multiple features (from an .mgf file, e.g.
  feature_id-keyed MZmine3 output) need a single overview of fragment-ion overlap
  and neutral-loss sharing for a feature or feature group of interest.
when_to_use_negative:
- The goal is a global pairwise similarity score (ms2deepscore, modified cosine, spec2vec)
  for an entire dataset rather than a direct visual inspection of fragment overlap
  for a selected feature group
- The spectra have no feature_id metadata key and cannot easily be renamed via the
  matchms Spectrum module, so the dashboard cannot tie rows to features
- The user needs a quantitative ML similarity (ms2deepscore) backbone instead of a
  qualitative fragmentation-overlap map
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Interactive Python dashboard whose fragmentation overview (fragmap) component
    renders the neutral-loss and fragment-intensity heatmap
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the MS/MS spectral data in .mgf format and provides the Spectrum module
    used to add/rename the feature_id metadata key
  repo: https://matchms.readthedocs.io/en/latest/
- name: plotly
  role: Rendering library for the fragmap heatmap with intensity color gradient and
    constant-colored neutral-loss blocks
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:0a9a1ece1bb1887a0ca9fb82c61ae1109103ab39a5fbadd0773b825a3be134de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - matchms
  - plotly
  - specXplore
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Neutral-loss annotation visualization

## Summary

Builds a fragmentation overview (fragmap) heatmap in specXplore/plotly that aligns MS/MS spectra of a selected feature group along one axis and factorized m/z fragment bins along the other, filling cells with fragment intensities and overlaying neutral losses as constant-colored blocks so shared fragmentation patterns become readable.

## When to use

Use when MS/MS spectra of multiple features (from an .mgf file, e.g. feature_id-keyed MZmine3 output) need a single overview of fragment-ion overlap and neutral-loss sharing for a feature or feature group of interest.

## When NOT to use

- The goal is a global pairwise similarity score (ms2deepscore, modified cosine, spec2vec) for an entire dataset rather than a direct visual inspection of fragment overlap for a selected feature group
- The spectra have no feature_id metadata key and cannot easily be renamed via the matchms Spectrum module, so the dashboard cannot tie rows to features
- The user needs a quantitative ML similarity (ms2deepscore) backbone instead of a qualitative fragmentation-overlap map

## Inputs

- .mgf file with MS/MS spectra (feature identifier key must be "feature_id")
- Selected feature or feature group of interest (subset of spectra with precursor m/z and peak tuples)

## Outputs

- Fragmentation overview map (fragmap): a 2D heatmap of fragment intensities per spectrum, with factorized fragment-bin columns and constant-colored neutral-loss overlay blocks
- Readable single-view assessment of which fragment ions and neutral losses are shared (or absent) among the selected spectra

## How to apply

Import the MS/MS spectra of the selected feature group from the .mgf file using matchms, retaining precursor m/z and peak tuples per feature. Bin the mass-to-charge ratios across all selected spectra, sort the bins in ascending order, and factorize them so crowded regions of the m/z axis are split and unused white space between distant fragments is removed. Assemble a 2D matrix with one row per spectrum (y-axis alignment) and factorized fragment bins as columns, fill cells with fragment intensity values, and render with plotly using a color gradient for intensity. Overlay neutral losses as constant-colored blocks inside the same heatmap so shared fragmentation patterns among the selected spectra become visible in one overview.

## Related tools

- **specXplore** (Interactive Python dashboard whose fragmentation overview (fragmap) component renders the neutral-loss and fragment-intensity heatmap) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the MS/MS spectral data in .mgf format and provides the Spectrum module used to add/rename the feature_id metadata key) — https://matchms.readthedocs.io/en/latest/
- **plotly** (Rendering library for the fragmap heatmap with intensity color gradient and constant-colored neutral-loss blocks) — https://github.com/kevinmildau/specXplore

## Examples

```
from matchms.importing import load_from_mgf; spectra = load_from_mgf("features.mgf")  # then run the specXplore importing pipeline and open the dashboard fragmap view
```

## Evaluation signals

- The fragmap renders successfully in the dashboard and displays fragment intensity cells for every selected spectrum row
- Shared fragment ions appear as aligned cells across spectra rows, making overlaps (or their absence) readable in a single overview
- Neutral losses are drawn as constant-colored blocks inside the heatmap, distinct from the intensity gradient of fragments
- The m/z axis shows no large white-space gaps between distant fragments (factorized, sorted bins) and crowded regions are split rather than merged
- Observed overlap patterns are consistent with the pairwise similarity scores (e.g. ms2deepscore) computed for the same feature group

## Limitations

- The current version of specXplore works on macOS and Linux but fails on Windows; macOS arm64 users may get unreliable ms2deepscore results due to issue 199
- Scalability beyond ~5000 features is untested; the dashboard is expected to face difficulties for larger data sets and liberal settings
- The fragmap is complementary to the pairwise similarity matrix; on its own it is a visual inspection tool, not a substitute for spectral similarity scoring

## Evidence

- [results] Bin the mass-to-charge ratios across all selected spectra, sort the bins in ascending order, and factorize the sorted bins so crowded regions of the m/z axis are split and unused white space between distant fragments is removed.: [evidence span withheld — anchor status: not_found]
- [intro] a fragmentation overview map enabling users to investigate fragmentation overlaps across multiple spectra directly from within the dashboard: "specXplore provides means for (a) investigating the raw pairwise similarity matrix directly, (b) investigating the fragmentation overlaps across multiple spectra"
- [readme] specXplore currently requires a .mgf formatted file with MS/MS spectral data.: [evidence span withheld — anchor status: external_doc]
- [readme] Feature lists should always contain some form of feature identifier, and specXplore expects the feature identifier key to be "feature_id".: [evidence span withheld — anchor status: external_doc]
- [methods] Spectral data are imported into Python using matchms: "Spectral data are imported into Python using matchms"
- [discussion] We expect that the dashboard will scale well to 5000 features but face difficulties for larger data sets and liberal settings.: "We expect that the dashboard will scale well to 5000 features but face difficulties for larger data sets and liberal settings"
