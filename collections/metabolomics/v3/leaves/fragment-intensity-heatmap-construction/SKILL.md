---
name: fragment-intensity-heatmap-construction
description: Use when after selecting a feature or feature group of interest (e.g.
  from the t-SNE overview or k-medoid clustering in the specXplore dashboard), you
  need a single-figure overview of how fragment ions and neutral losses overlap across
  the multiple MS/MS spectra of that group.
when_to_use_negative:
- When the goal is to inspect the global pairwise spectral similarity structure (e.g.
  a full ms2deepscore similarity matrix or t-SNE overview of the whole feature set)
  rather than the fragmentation content of a small local group of spectra.
- When no MS/MS peak lists are available — e.g. only MS1 feature intensities or a
  feature table without .mgf-derived fragment spectra — since the heatmap cells must
  be filled with fragment intensities from the spectra.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Dashboard application in which the fragmentation overview (fragmap) view is
    rendered as one of its complementary interactive visualizations for a selected
    feature or feature group.
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the MS/MS spectra of the selected feature group from the .mgf file
    into Python, providing the per-spectrum precursor and peak data used to build
    the heatmap.
- name: plotly
  role: Renders the 2D fragment-intensity matrix as an interactive heatmap with an
    intensity color gradient, in which neutral-loss blocks are overlaid.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:e7cf8cbe8046f921ba0f348cd6ce69999d58f451a98b0bacffdd5e11b1c23152
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - matchms
  - plotly
  - specXplore
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fragment-intensity-heatmap-construction

## Summary

Construct a fragmentation-overlap (fragmap) heatmap that renders, per spectrum of a selected feature or feature group, binned and factorized fragment m/z positions colored by fragment intensity, with neutral-loss blocks overlaid so shared fragmentation patterns become readable across all selected spectra.

## When to use

Use when, after selecting a feature or feature group of interest (e.g. from the t-SNE overview or k-medoid clustering in the specXplore dashboard), you need a single-figure overview of how fragment ions and neutral losses overlap across the multiple MS/MS spectra of that group.

## When NOT to use

- When the goal is to inspect the global pairwise spectral similarity structure (e.g. a full ms2deepscore similarity matrix or t-SNE overview of the whole feature set) rather than the fragmentation content of a small local group of spectra.
- When no MS/MS peak lists are available — e.g. only MS1 feature intensities or a feature table without .mgf-derived fragment spectra — since the heatmap cells must be filled with fragment intensities from the spectra.

## Inputs

- MS/MS spectra of a selected feature or feature group of interest, as imported from an .mgf file by matchms (each spectrum with precursor m/z and peak (m/z, intensity) tuples)
- Neutral-loss values associated with the selected spectra for the constant-colored overlay blocks
- A specXplore session data object produced by the importing pipeline, from which the selected spectra are drawn in the dashboard

## Outputs

- A 2D fragmentation-overlap heatmap (fragmap) with one row per spectrum and factorized fragment bins as columns, cells colored by fragment intensity via a plotly color gradient
- Constant-colored neutral-loss blocks overlaid within the same heatmap, making shared fragmentation patterns across spectra visible

## How to apply

Take the MS/MS spectra of the selected feature group as extracted from the .mgf file by matchms, each carrying its precursor m/z and its peak (m/z, intensity) tuples. Bin the mass-to-charge ratios across all selected spectra, sort the bins in ascending order, and factorize the sorted bins so that crowded m/z regions are split and unused white space between distant fragments is removed; the factorized bins become the heatmap columns while spectra form the rows. Fill each cell with the corresponding fragment intensity and render the 2D matrix with plotly using a color gradient for intensity, so the y-axis aligns one spectrum per row. Overlay neutral losses as constant-colored blocks inside the same heatmap so that shared fragmentation patterns, or their absence, are visible in one overview. This dashboard view complements the pairwise similarity matrix inspection: the overlaps it exposes should be consistent with the ms2deepscore / modified cosine / spec2vec scores computed for that feature group.

## Related tools

- **specXplore** (Dashboard application in which the fragmentation overview (fragmap) view is rendered as one of its complementary interactive visualizations for a selected feature or feature group.) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the MS/MS spectra of the selected feature group from the .mgf file into Python, providing the per-spectrum precursor and peak data used to build the heatmap.)
- **plotly** (Renders the 2D fragment-intensity matrix as an interactive heatmap with an intensity color gradient, in which neutral-loss blocks are overlaid.)

## Examples

```
from matchms.importing import load_from_mgf; spectra = list(load_from_mgf('features.mgf')); # then bin+factorize m/z values of the selected feature group and render with plotly as in the specXplore importing pipeline
```

## Evaluation signals

- The rendered fragmap displays one row per selected spectrum and its column axis consists of factorized, sorted fragment bins with the crowded-m/z-region splitting applied (no large blank gaps between distant fragments).
- Cell intensities follow a color gradient and correspond to the fragment intensities of the respective spectra extracted from the .mgf input via matchms.
- Neutral losses appear as constant-colored blocks within the same heatmap, so that identical fragmentation patterns across spectra can be identified at a glance.
- The overlaps (or their absence) visible in the fragmap are consistent with the pairwise similarity scores (ms2deepscore, modified cosine, spec2vec) computed for the same feature group.

## Limitations

- The source card describing the fragmap workflow cites specXplore as the implementing tool, but the article itself names the component only as a 'fragmentation overview map'; details of the binning/factorization and neutral-loss overlay come from the skill card and should be validated against the repository code.
- The fragmap is meaningful only for a selected local group of spectra; it does not summarize similarity structure over the full dataset, for which the t-SNE overview and partial network drawings are the intended views.
- The pipeline presupposes a .mgf file with a 'feature_id' key per spectrum; specXplore currently requires this format and fails on Windows, so spectra must be prepared accordingly (e.g. via MZmine3 export and matchms metadata renaming).

## Evidence

- [methods] Fragmap construction from .mgf spectra via matchms: "Spectral data are imported into Python using matchms"
- [intro] Fragmap is one of specXplore's complementary dashboard views: "specXplore provides means for (a) investigating the raw pairwise similarity matrix directly, (b) investigating the fragmentation overlaps across multiple spectra"
- [readme] matchms as the spectral import path for the pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] specXplore package includes fragmentation overview maps: [evidence span withheld — anchor status: external_doc]
