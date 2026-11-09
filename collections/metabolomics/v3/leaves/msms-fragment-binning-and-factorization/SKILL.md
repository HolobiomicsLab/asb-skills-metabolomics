---
name: msms-fragment-binning-and-factorization
description: Use when comparing MS/MS spectra of several features from an .mgf file
  to see which fragment m/z values and neutral losses they share in one heatmap view.
when_to_use_negative:
- You only need pairwise similarity scores (ms2deepscore, modified cosine, spec2vec)
  rather than a fragment-level overlap view — those are computed separately in the
  importing pipeline.
- Input spectra are not in .mgf format; specXplore currently requires .mgf-formatted
  MS/MS data (convert e.g. via MZmine3 export first).
- Only one spectrum is of interest — with a single spectrum there is no cross-spectral
  overlap to display and a fragmap adds nothing over a plain spectrum plot.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Imports the .mgf MS/MS spectra into Python and provides Spectrum objects used
    to extract fragment peaks and intensities for binning
  repo: https://matchms.readthedocs.io/en/latest/
- name: plotly
  role: Renders the binned/factorized intensity matrix as the interactive fragmentation
    overview heatmap with a color gradient for intensity
- name: specXplore
  role: Dashboard in which the resulting fragmentation overview map is displayed alongside
    t-SNE, network, and similarity heatmap views
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:8c42dda5bb34415633bf9d2a7cab14a9faf9b3b6a0edeab589f29507917b6cfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matchms
  - plotly
  - specXplore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msms-fragment-binning-and-factorization

## Summary

Bins the mass-to-charge ratios of MS/MS fragments from multiple spectra, sorts the bins ascending, and factorizes them so crowded m/z regions are split and empty space between distant fragments is removed — the data-preparation step behind specXplore's fragmentation overview (fragmap) heatmap.

## When to use

Use when comparing MS/MS spectra of several features from an .mgf file to see which fragment m/z values and neutral losses they share in one heatmap view.

## When NOT to use

- You only need pairwise similarity scores (ms2deepscore, modified cosine, spec2vec) rather than a fragment-level overlap view — those are computed separately in the importing pipeline.
- Input spectra are not in .mgf format; specXplore currently requires .mgf-formatted MS/MS data (convert e.g. via MZmine3 export first).
- Only one spectrum is of interest — with a single spectrum there is no cross-spectral overlap to display and a fragmap adds nothing over a plain spectrum plot.

## Inputs

- .mgf file with MS/MS spectra (precursor m/z and peak tuples per feature), e.g. produced by MZmine3 export
- Selected feature group of interest (feature identifiers keyed as 'feature_id')

## Outputs

- 2D intensity matrix (spectra × factorized fragment bins) rendered as a plotly fragmap heatmap
- Neutral-loss overlays rendered as constant-colored blocks inside the same heatmap

## How to apply

Load the MS/MS spectra of the selected feature group from the .mgf file into Python with matchms, keeping each spectrum's precursor m/z and peak tuples. Collect all fragment mass-to-charge ratios across the selected spectra and bin them; sort the resulting bins in ascending order, then factorize (assign integer indices to) the sorted bins so that crowded regions of the m/z axis are split apart and unused white space between distant fragments is collapsed away. Assemble a 2D matrix with one row per spectrum and the factorized fragment bins as columns, filling each cell with the fragment's intensity. Render with plotly on a color-gradient heatmap, aligning spectra on the y-axis, and overlay neutral losses as constant-colored blocks so shared fragmentation patterns are visible. The rationale is that raw m/z values waste space and merge crowded regions, whereas the factorized bins make spectral overlap (or its absence) readable in a single overview consistent with the pairwise similarity scores computed for the same feature group.

## Related tools

- **matchms** (Imports the .mgf MS/MS spectra into Python and provides Spectrum objects used to extract fragment peaks and intensities for binning) — https://matchms.readthedocs.io/en/latest/
- **plotly** (Renders the binned/factorized intensity matrix as the interactive fragmentation overview heatmap with a color gradient for intensity)
- **specXplore** (Dashboard in which the resulting fragmentation overview map is displayed alongside t-SNE, network, and similarity heatmap views) — https://github.com/kevinmildau/specXplore

## Examples

```
from specxplore.importing import Importer; Importer().import_data(mgf_file='features.mgf', models_folder='models/pos')
```

## Evaluation signals

- Every spectrum in the selected feature group appears as one row of the fragmap, aligned on the y-axis.
- Fragment bins are integer-indexed (factorized) and sorted ascending, so no two crowded m/z regions share a column and gaps between distant fragments are removed.
- Cell intensities fall within the intensity range of the original spectra, and a color gradient maps them legibly.
- Neutral losses appear as distinct constant-colored blocks that do not follow the intensity color scale.
- Shared fragment m/z values across spectra line up in the same column, so overlapping spectra visibly agree where ms2deepscore/modifed-cosine assign high similarity for that feature group.

## Limitations

- The pipeline (and hence this fragmap view) currently works on macOS and Linux but fails on Windows; ms2deepscore similarity results may additionally be unreliable on macOS arm64 (issue 199), affecting the companion similarity scores shown with the fragmap.
- specXplore requires .mgf-formatted MS/MS data with a feature identifier key named 'feature_id'; files with other keys must be renamed via matchms or a text editor first.
- The dashboard is expected to scale well to ~5000 features but may face difficulties for larger datasets with liberal settings.

## Evidence

- [methods] Binning and factorization of fragment m/z values: [evidence span withheld — anchor status: not_found]
- [methods] matchms import of .mgf spectra: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [intro] Dashboard provides fragmentation-overlap views: "specXplore provides means for (a) investigating the raw pairwise similarity matrix directly, (b) investigating the fragmentation overlaps across multiple spectra, and (c) inspecting any joined-in"
- [readme] README: .mgf requirement and feature_id key: [evidence span withheld — anchor status: external_doc]
- [readme] README: MZmine3 export path: [evidence span withheld — anchor status: external_doc]
