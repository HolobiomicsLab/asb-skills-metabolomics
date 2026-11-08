---
name: body-site-volatilome-visualization
description: Use when deconvoluted GC-MS feature tables carry per-body-location coordinates/intensities
  and the goal is to render compound spatial distributions on a 3D human body model
  (e.g. rebuilding volatilome body maps like Figure 2f–i).
when_to_use_negative:
- Input is a raw GC-MS instrument file (e.g. mzML/NetCDF) that has not yet been auto-deconvoluted
  through MSHub/GNPS — deconvolution must precede body-map rendering
- Feature tables lack per-body-location sampling coordinates/intensities, so no spatial
  mapping is possible
- The analysis goal is spectral annotation or molecular networking itself rather than
  spatial visualization of already-annotated features
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Source of the GC-MS deconvolution feature tables (via MSHub/GNPS auto-deconvolution
    and molecular networking workflows) that are filtered and visualized
  repo: github.com/bittremieux/GNPS_GC
- name: '''ili'
  role: Interactive 3D human body model used to render each compound's per-body-location
    intensity distribution
- name: pandas
  role: Loading, merging, and threshold-filtering the headspace and liquid-injection
    feature tables (balance ≥ 50%, cosine > 0.9)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:2adc2217ebeeea442d6aec893809a704d027951c3847609d915f06cb7ec9ae9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - '''ili'
  - GNPS
  - pandas
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# body-site-volatilome-visualization

## Summary

Maps deconvoluted GC-MS feature tables (headspace and liquid-injection, annotated via GNPS/MSHub) onto a 3D human body model in 'ili to visualize spatial distributions of skin volatiles, e.g. squalene, hexanoic acid, globulol, and phenylene dibenzoate across body locations.

## When to use

Use when deconvoluted GC-MS feature tables carry per-body-location coordinates/intensities and the goal is to render compound spatial distributions on a 3D human body model (e.g. rebuilding volatilome body maps like Figure 2f–i).

## When NOT to use

- Input is a raw GC-MS instrument file (e.g. mzML/NetCDF) that has not yet been auto-deconvoluted through MSHub/GNPS — deconvolution must precede body-map rendering
- Feature tables lack per-body-location sampling coordinates/intensities, so no spatial mapping is possible
- The analysis goal is spectral annotation or molecular networking itself rather than spatial visualization of already-annotated features

## Inputs

- GNPS GC-MS deconvolution feature table (headspace analysis) with body-location coordinates
- GNPS GC-MS deconvolution feature table (liquid-injection analysis) with body-location coordinates
- Compound identifiers/names of target volatiles (e.g. squalene, hexanoic acid, globulol, phenylene dibenzoate)

## Outputs

- 3D body-map images of per-compound volatile intensity distributions (body_volatilome_maps.png, Figure 2f–i style panels)
- Filtered feature table retaining only balance ≥ 50% and cosine > 0.9 annotations (filtered_feature_table.csv)

## How to apply

Load the headspace and liquid-injection GNPS deconvolution feature tables from the GNPS_GC release (github.com/bittremieux/GNPS_GC) into pandas DataFrames, then merge them into a single combined feature table keyed on deconvoluted feature identity. Filter features using the paper's quality thresholds: balance score ≥ 50% (which quantifies the reproducibility of fragmentation patterns across all samples) and spectral cosine similarity > 0.9. Extract per-body-location intensity profiles for target compounds such as squalene, hexanoic acid, globulol, and phenylene dibenzoate from the filtered table. Render each compound's intensity distribution on the 3D human body model in 'ili to reproduce the Figure 2f–i panels, and export the body-map images (body_volatilome_maps.png) together with the filtered_feature_table.csv.

## Related tools

- **GNPS** (Source of the GC-MS deconvolution feature tables (via MSHub/GNPS auto-deconvolution and molecular networking workflows) that are filtered and visualized) — github.com/bittremieux/GNPS_GC
- **'ili** (Interactive 3D human body model used to render each compound's per-body-location intensity distribution)
- **pandas** (Loading, merging, and threshold-filtering the headspace and liquid-injection feature tables (balance ≥ 50%, cosine > 0.9))

## Examples

```
df = pd.concat([pd.read_csv('headspace_features.csv'), pd.read_csv('liquid_injection_features.csv')]); filtered = df[(df['balance'] >= 50) & (df['cosine'] > 0.9)]; filtered.to_csv('filtered_feature_table.csv', index=False)
```

## Evaluation signals

- The merged feature table reduces to annotations passing balance ≥ 50% AND spectral cosine similarity > 0.9; check no rows below either threshold remain
- Squalene should appear at all sampled body locations but with lower intensity on the feet, matching the published spatial distribution
- The rendered body maps visually correspond to the paper's Figure 2f–i panels for squalene, hexanoic acid, globulol, and phenylene dibenzoate
- filtered_feature_table.csv retains deconvoluted feature identity keys and per-body-location intensity columns for both headspace and liquid-injection sources

## Limitations

- Spatial resolution is limited to the sampled body locations in the study; locations not sampled cannot be visualized
- Visualizations are only as reliable as the underlying annotations — the balance score and cosine > 0.9 filters are needed to remove irreproducible fragment assignments
- The GNPS_GC repository is archived, so feature-table availability depends on the archived release

## Evidence

- [abstract] balance score definition: "We introduce a “balance score” that quantifies the reproducibility of fragmentation patterns across all samples."
- [abstract] first molecular spatial map of the human volatilome: "We demonstrate the utility of the platform with breathomics analysis applied to the early detection of oesophago-gastric cancer, and by creating the first molecular spatial map of the human"
- [readme] GNPS_GC companion repository: [evidence span withheld — anchor status: not_found]
- [abstract] GNPS molecular networking platform role: "workflows to enable the community to store, process, share, annotate, compare, and perform molecular networking of GC-MS data within the Global Natural Product Social (GNPS) Molecular Networking"
