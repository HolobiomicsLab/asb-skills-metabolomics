---
name: 3d-surface-model-coordinate-mapping
description: Use when per-feature intensities with sample x/y/z coordinates and spot
  radii from Optimus must be visualized on a 3D STL model or 2D image in 'ili, or
  when a configured map view must be saved and reproduced via a cartographical snapshot.
when_to_use_negative:
- The task is upstream LC–MS feature detection, quantification, filtering or annotation
  — those are Optimus workflow steps; this skill only handles the visualization/spatial-mapping
  stage.
- The data are not spatially mapped (no x/y/z sample coordinates or no 3D model/2D
  image of the object) — 'ili cannot place features without a spatial context.
- The goal is inspecting extracted ion chromatograms, MS peaks or MS2 spectra per
  feature — use OptimusViewer instead of 'ili.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: '''ili'
  role: Interactive web application in which the feature intensities, x/y/z coordinates
    and spot radii are mapped as spots onto the 3D STL model / 2D PNG background;
    supports spot Size, hotspot quantile, Linear/Logarithmic scales, and cartographical
    snapshot capture with Ctrl/Cmd+E.
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: Upstream KNIME/OpenMS workflow that performs feature detection, quantification
    and spatial-map creation, producing the 'ili_input.csv feature-intensity table
    consumed by this skill.
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:d5f07c9d4928fb2b4bcedf3220b8f80500c5edf34f7701b66364bca80c064cad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - Optimus
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 24
---

# 3d-surface-model-coordinate-mapping

## Summary

Maps spatially resolved LC–MS feature intensities (the Optimus-produced 'ili_input.csv) as colored spots onto a binary STL 3D surface model or PNG 2D background in the 'ili app, and captures a cartographical snapshot (JSON of selected map, colormap and view position) that reproduces the exact map view.

## When to use

Use when per-feature intensities with sample x/y/z coordinates and spot radii from Optimus must be visualized on a 3D STL model or 2D image in 'ili, or when a configured map view must be saved and reproduced via a cartographical snapshot.

## When NOT to use

- The task is upstream LC–MS feature detection, quantification, filtering or annotation — those are Optimus workflow steps; this skill only handles the visualization/spatial-mapping stage.
- The data are not spatially mapped (no x/y/z sample coordinates or no 3D model/2D image of the object) — 'ili cannot place features without a spatial context.
- The goal is inspecting extracted ion chromatograms, MS peaks or MS2 spectra per feature — use OptimusViewer instead of 'ili.

## Inputs

- 'ili_input.csv — Optimus feature-intensity table (from mzML-derived output) with feature intensities, sample x/y/z coordinates and spot radii
- Binary STL 3D surface model (or PNG 2D background image) of the mapped object
- 5-column coordinates file (Sample, x, y, z, radius)

## Outputs

- Interactive 2D/3D molecular maps of detected features visualized as spots on the surface model in 'ili
- cartographical_snapshot.json — JSON file storing all visualization settings (selected map, colormap, view position)

## How to apply

Load the Optimus feature-intensity table ('ili_input.csv, produced from the mzML-derived output) containing feature intensities, sample x/y/z coordinates and spot radii into 'ili alongside the binary STL 3D model (or PNG 2D background) and the 5-column coordinates file (Sample, x, y, z, radius). Browse per-feature molecular maps and set the radii 'Size' factor in the 'Spots' menu so neighboring spots do not overlap. For maps dominated by a few high-intensity spots, reduce the 'hotspot quantile' value or switch the scale from 'Linear' to 'Logarithmic'. Fine-tune parameters in the '2D' or '3D' tab of the 'ili sidebar and navigate the view (scroll to zoom, left-drag to move). Capture the cartographical snapshot with 'Ctrl'/'Cmd' + 'E', which writes a JSON file recording the selected map, colormap and view position to the downloads directory; rename/keep it as cartographical_snapshot.json. Validate by dragging the JSON back into 'ili with the same model and 'ili_input.csv loaded and confirming the identical view is restored.

## Related tools

- **'ili** (Interactive web application in which the feature intensities, x/y/z coordinates and spot radii are mapped as spots onto the 3D STL model / 2D PNG background; supports spot Size, hotspot quantile, Linear/Logarithmic scales, and cartographical snapshot capture with Ctrl/Cmd+E.) — https://github.com/ili-toolbox/ili
- **Optimus** (Upstream KNIME/OpenMS workflow that performs feature detection, quantification and spatial-map creation, producing the 'ili_input.csv feature-intensity table consumed by this skill.) — https://github.com/MolecularCartography/Optimus

## Evaluation signals

- Dragging the saved cartographical_snapshot.json back into 'ili with the same 2D/3D model and 'ili_input.csv loaded restores exactly the same view (selected map, colormap, view position).
- The snapshot file is valid JSON and contains the recorded visualization settings (selected map, colormap, view position).
- Spot radii 'Size' factor in the 'Spots' menu is set so neighboring spots do not overlap on the model surface.
- For hotspot-dominated maps, reducing the 'hotspot quantile' or switching 'Linear'→'Logarithmic' makes the spatial pattern legible rather than saturated.

## Limitations

- A cartographical snapshot only reproduces the exact view when the same 2D/3D model and the same 'ili_input.csv are loaded alongside it; mismatched inputs break reproduction.
- The workflow requires a coordinate file and a model of the mapped object — 'ili cannot visualize features lacking spatial coordinates or a surface to map onto.
- 'ili is a visualization tool only; feature detection, quantification, filtering and annotation must be completed upstream with Optimus.

## Evidence

- [abstract] Cartographical snapshots are introduced as shareable, reloadable files that reproduce a map's visualization settings: "we introduce cartographical snapshots: files that describe a particular map and visualization settings, and that can be shared and loaded to reproduce the visualization."
- [intro] Enable spatial mapping / molecular cartography of detected molecular features in 2D and 3D: "for spatial mapping of detected molecular features in 2D and 3D using the [`ili app"
- [readme] 'ili visualizes spatial data on an image or a 3D model: "It is a web-application for interactive visualization of spatial data mapped either on an image or a 3D model, also developed by Alexandrov Team."
- [intro] Creating spatial maps of detected features for 'ili visualization: "14. Creating spatial maps of detected features that can be visualized in [`ili app"
