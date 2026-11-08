---
name: feature-intensity-colormap-scaling
description: Use when a molecular map of feature intensities in 'ili is dominated
  by a few high-intensity spots, neighboring spots overlap, or the color range obscures
  spatial gradients in the intensity data.
when_to_use_negative:
- The input is a raw mzML or an Optimus *.db file rather than the 'ili_input.csv table
  — run feature detection/quantification and spatial-map export in Optimus first
- The data is not spatially mapped (no sample x/y/z coordinates); 'ili visualization
  and its spot sizing/scaling controls do not apply
- You need LC–MS feature detection, filtering, or annotation — those are Optimus workflow
  steps, not 'ili rendering adjustments
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3520
tools:
- name: '''ili'
  role: Web application used to visualize per-feature molecular maps on 2D images
    or 3D models, where the spot 'Size' factor, 'hotspot quantile', and Linear/Logarithmic
    scale are adjusted
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: KNIME/OpenMS workflow that generates the 'ili_input.csv feature-intensity
    table with sample x/y/z coordinates and spot radii that 'ili renders
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:e76c6e3372aa81161e4245c4f08221e91d047667d7464bb2cd6ffafa1393f0e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - Optimus
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# feature-intensity-colormap-scaling

## Summary

Tuning how per-feature molecular intensities are rendered in 'ili — adjusting the spot 'Size' radius factor, the 'hotspot quantile', and the Linear/Logarithmic scale — so that maps of LC–MS feature intensities mapped on 2D images or 3D STL models remain legible and spatially interpretable.

## When to use

Use when a molecular map of feature intensities in 'ili is dominated by a few high-intensity spots, neighboring spots overlap, or the color range obscures spatial gradients in the intensity data.

## When NOT to use

- The input is a raw mzML or an Optimus *.db file rather than the 'ili_input.csv table — run feature detection/quantification and spatial-map export in Optimus first
- The data is not spatially mapped (no sample x/y/z coordinates); 'ili visualization and its spot sizing/scaling controls do not apply
- You need LC–MS feature detection, filtering, or annotation — those are Optimus workflow steps, not 'ili rendering adjustments

## Inputs

- 'ili_input.csv (per-feature intensity table produced by Optimus from mzML-derived output, with feature intensities, sample x/y/z coordinates, and spot radii)
- Binary STL 3D model or PNG 2D background image
- 5-column coordinates file (Sample, x, y, z, radius)

## Outputs

- Adjusted 'ili map view with per-feature spot radii, hotspot quantile, and Linear/Logarithmic color scale set for legible visualization
- Visualization settings that can be captured in a 'ili cartographical snapshot (JSON via Ctrl/Cmd+E)

## How to apply

Load the Optimus-produced 'ili_input.csv (feature intensities, sample x/y/z coordinates, spot radii) into 'ili alongside the STL 3D model or PNG 2D background. In the 'Spots' menu, set the radii 'Size' factor so neighboring spots do not overlap on the 2D/3D model. For maps dominated by a few high-intensity spots, reduce the 'hotspot quantile' value to compress the upper intensity range, or switch the scale from 'Linear' to 'Logarithmic' to spread the dynamic range of low-intensity features. Fine-tune remaining mapping parameters in the '2D' or '3D' tab of the 'ili sidebar, navigating with scroll-to-zoom and left-drag to pan.

## Related tools

- **'ili** (Web application used to visualize per-feature molecular maps on 2D images or 3D models, where the spot 'Size' factor, 'hotspot quantile', and Linear/Logarithmic scale are adjusted) — https://github.com/ili-toolbox/ili
- **Optimus** (KNIME/OpenMS workflow that generates the 'ili_input.csv feature-intensity table with sample x/y/z coordinates and spot radii that 'ili renders) — https://github.com/MolecularCartography/Optimus

## Evaluation signals

- Neighboring spots on the 2D/3D model no longer overlap after adjusting the radii 'Size' factor
- Spatial gradients in low-intensity regions become distinguishable after switching the scale from 'Linear' to 'Logarithmic' or reducing the 'hotspot quantile'
- The chosen colormap and scale are recorded in the cartographical snapshot JSON and reproduce the identical view when the JSON is dragged back into 'ili
- The rendered map corresponds to intensities in 'ili_input.csv for the selected feature (spot colors track the table values)

## Limitations

- Logarithmic scaling and hotspot-quantile compression change perceived intensity relationships; the chosen setting must be recorded in a cartographical snapshot to keep the view reproducible
- Spot radius sizing is a trade-off: larger spots improve visibility but cause neighboring spots to overlap on dense 2D/3D models
- 'ili visualization requires the same 2D/3D model and 'ili_input.csv to be loaded when restoring settings from a snapshot

## Evidence

- [readme] spot sizing adjustment: [evidence span withheld — anchor status: external_doc]
