---
name: multi-spectrum-fragmentation-overlap-assessment
description: Use when several MS/MS spectra from an .mgf file (e.g. a feature group
  of interest) must be compared for shared fragment m/z bins and neutral losses, beyond
  what a single mirror plot or pairwise similarity score can reveal.
when_to_use_negative:
- You only need a single pairwise spectrum comparison or a mirror plot for one pair
  of spectra — the fragmap is for multi-spectrum overlap inspection.
- You are inspecting the global similarity structure; the t-SNE overview network or
  pairwise similarity matrix views are the intended entry points for that.
- The .mgf file lacks the 'feature_id' metadata key specXplore expects, and no matchms-based
  renaming has been done.
- Your data exceeds roughly 5000 features, where dashboard scaling is untested and
  performance difficulties are anticipated.
tools:
- name: specXplore
  role: Python dashboard whose importing pipeline and dashboard module produce and
    render the fragmentation overview map for the selected feature group
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the .mgf MS/MS spectra into Python and handles spectrum objects (including
    the 'feature_id' metadata key) for binning and fragmap assembly
- name: plotly
  role: Renders the 2D fragmap heatmap with an intensity color gradient and overlaid
    neutral-loss blocks
- name: ms2deepscore
  role: Provides the pairwise similarity scores against which the fragmap's visualized
    overlaps are cross-checked for consistency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:41fc73512f18e60f319acdebfc6ca2664ee30ae81e8dce44ab6ecba345cf2fd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - matchms
  - ms2deepscore
  - plotly
  - specXplore
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-spectrum-fragmentation-overlap-assessment

## Summary

Builds a 2D fragmap heatmap in specXplore that aligns all MS/MS spectra of a selected feature group along the y-axis against factorized fragment m/z bins on the x-axis, filling cells with fragment intensities and overlaying neutral losses so spectral overlaps (or their absence) become visible in a single overview.

## When to use

Use when several MS/MS spectra from an .mgf file (e.g. a feature group of interest) must be compared for shared fragment m/z bins and neutral losses, beyond what a single mirror plot or pairwise similarity score can reveal.

## When NOT to use

- You only need a single pairwise spectrum comparison or a mirror plot for one pair of spectra — the fragmap is for multi-spectrum overlap inspection.
- You are inspecting the global similarity structure; the t-SNE overview network or pairwise similarity matrix views are the intended entry points for that.
- The .mgf file lacks the 'feature_id' metadata key specXplore expects, and no matchms-based renaming has been done.
- Your data exceeds roughly 5000 features, where dashboard scaling is untested and performance difficulties are anticipated.

## Inputs

- MS/MS spectra in .mgf format from a feature list where the feature identifier key is 'feature_id' (as produced by e.g. MZmine3 export)
- A selected feature or feature group of interest with precursor m/z and peak tuples per spectrum
- A specXplore session data object processed via the specXplore importing pipeline

## Outputs

- Interactive fragmentation overview map (fragmap) rendered with plotly, one row per spectrum against factorized fragment m/z bins
- Per-cell fragment intensity values encoded via a color gradient
- Constant-colored neutral-loss blocks overlaid on the heatmap

## How to apply

Import the MS/MS spectra of the selected feature group from the .mgf file into Python using matchms, keeping precursor m/z and peak tuples per feature. Bin the mass-to-charge ratios across all selected spectra, sort the bins in ascending order, and factorize the sorted bins so crowded m/z regions are split and unused white space between distant fragments is removed. Assemble a 2D matrix with one row per spectrum (y-axis alignment) and the factorized fragment bins as columns, filling cells with fragment intensity values, and render it with plotly using a color gradient for intensity. Overlay neutral losses as constant-colored blocks inside the same heatmap so shared fragmentation patterns stand out. Validate the view by checking that the fragmap makes spectral overlaps among the selected spectra readable in a single overview, consistent with the pairwise ms2deepscore similarity scores computed for that feature group.

## Related tools

- **specXplore** (Python dashboard whose importing pipeline and dashboard module produce and render the fragmentation overview map for the selected feature group) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the .mgf MS/MS spectra into Python and handles spectrum objects (including the 'feature_id' metadata key) for binning and fragmap assembly)
- **plotly** (Renders the 2D fragmap heatmap with an intensity color gradient and overlaid neutral-loss blocks)
- **ms2deepscore** (Provides the pairwise similarity scores against which the fragmap's visualized overlaps are cross-checked for consistency)

## Examples

```
conda activate specxplore_environment; jupyter-notebook  # open demo.ipynb, set mgf filepath, run specXplore importing pipeline, then launch the dashboard session
```

## Evaluation signals

- The fragmap renders without error in the dashboard for the selected feature group.
- Shared fragmentation patterns (identical fragment m/z bins across rows) are visually identifiable via the intensity color gradient.
- Neutral losses appear as constant-colored blocks, making shared losses across spectra readable.
- The visualized overlaps are consistent with the pairwise ms2deepscore similarities computed for the same feature group.
- Factorized bins show no large white-space gaps, confirming unused m/z regions between distant fragments were removed.

## Limitations

- Requires a .mgf formatted file with MS/MS spectra; the feature identifier key must be 'feature_id', which may require renaming via matchms or a text editor.
- ms2deepscore (and hence its consistency check) only works well for spectra/metabolites well covered by or related to its training data; on macOS arm64, issue 199 for ms2deepscore can make similarity predictions unreliable without errors or warnings.
- The current version of specXplore works on macOS and Linux but fails on Windows.
- Performance is expected to be good up to ~5000 features but difficulties are anticipated for larger datasets and liberal settings.

## Evidence

- [other] Fragmap matrix assembly and neutral-loss overlay method: [evidence span withheld — anchor status: not_found]
- [intro] Fragmap purpose in the dashboard: "specXplore provides means for (a) investigating the raw pairwise similarity matrix directly, (b) investigating the fragmentation overlaps across multiple spectra, and (c) inspecting any joined-in"
- [methods] Import via matchms: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [readme] mgf requirement and feature_id key: [evidence span withheld — anchor status: external_doc]
- [readme] Windows support limitation: [evidence span withheld — anchor status: external_doc]
- [full_text] Dashboard component description: [evidence span withheld — anchor status: external_doc]
