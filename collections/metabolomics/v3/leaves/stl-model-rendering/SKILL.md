---
name: stl-model-rendering
description: Use when an Optimus-produced feature-intensity table ('ili_input.csv
  from mzML-derived output) with sample x/y/z coordinates and spot radii must be visualized
  as a molecular map on a 3D object represented by an STL model.
when_to_use_negative:
- The input is already a rendered molecular map or a cartographical snapshot JSON
  rather than a raw feature-intensity table
- The dataset lacks spatial coordinates (x/y/z plus radius per sample), so no spatial
  mapping onto a model is possible
- The object of interest has no available binary STL model or 2D image background
  onto which spots can be mapped
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: '''ili'
  role: Web-application used to interactively render the spatial map of detected features
    on an image or 3D model; spot size, colormap, hotspot quantile and view are adjusted
    in its sidebar
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: Produces the spatial-map input ('ili_input.csv) via its step 'Creating spatial
    maps of detected features that can be visualized in 'ili app'
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:3e43bceae087788ce54a201390ab761442ed7228823e3cdaa34bfcd16e2af117
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - Optimus
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# stl-model-rendering

## Summary

Render spatial molecular maps from LC–MS feature intensity data onto a binary STL 3D model (or PNG 2D background) in the 'ili app, adjusting spot size, colormap, scale and view position so molecular cartography matches the physical object's shape.

## When to use

Use when an Optimus-produced feature-intensity table ('ili_input.csv from mzML-derived output) with sample x/y/z coordinates and spot radii must be visualized as a molecular map on a 3D object represented by an STL model.

## When NOT to use

- The input is already a rendered molecular map or a cartographical snapshot JSON rather than a raw feature-intensity table
- The dataset lacks spatial coordinates (x/y/z plus radius per sample), so no spatial mapping onto a model is possible
- The object of interest has no available binary STL model or 2D image background onto which spots can be mapped

## Inputs

- 'ili_input.csv — feature intensity table produced from the mzML-derived Optimus output, containing feature intensities, sample x/y/z coordinates, and spot radii
- Binary STL 3D model of the mapped object (or PNG 2D background image)
- 5-column coordinates file (Sample, x, y, z, radius)

## Outputs

- Interactive 2D/3D molecular map with per-feature intensity rendered as spots positioned on the STL model surface
- cartographical_snapshot.json — JSON file recording visualization settings (selected map, colormap, view position), written to the downloads directory via Ctrl/Cmd+E

## How to apply

Load the Optimus feature-intensity table ('ili_input.csv) into 'ili together with the binary STL 3D model (or PNG 2D background) and the 5-column coordinates file (Sample, x, y, z, radius). Browse per-feature molecular maps and set the radii 'Size' factor in the 'Spots' menu so neighboring spots do not overlap. For maps dominated by a few high-intensity spots, reduce the 'hotspot quantile' value or switch the scale from 'Linear' to 'Logarithmic'. Fine-tune parameters in the '2D' or '3D' tab of the 'ili sidebar and navigate the view (scroll to zoom, left-drag to move); optionally capture the visualization state with Ctrl/Cmd+E into a cartographical snapshot JSON. The rationale is to map each feature's intensity at its sampled spatial position onto the surface of the digitized object, enabling 3D molecular cartography.

## Related tools

- **'ili** (Web-application used to interactively render the spatial map of detected features on an image or 3D model; spot size, colormap, hotspot quantile and view are adjusted in its sidebar) — https://github.com/ili-toolbox/ili
- **Optimus** (Produces the spatial-map input ('ili_input.csv) via its step 'Creating spatial maps of detected features that can be visualized in 'ili app') — https://github.com/MolecularCartography/Optimus

## Evaluation signals

- Dragging the saved cartographical_snapshot.json back into 'ili with the same 2D/3D model and 'ili_input.csv loaded restores exactly the same view (selected map, colormap, view position)
- The snapshot file is valid JSON containing the selected map, colormap, and view-position settings
- Spot radii ('Size' factor in the 'Spots' menu) are set such that neighboring spots do not overlap
- Feature intensities in hotspot-dominated maps are no longer clipped after reducing the hotspot quantile or switching the scale from 'Linear' to 'Logarithmic'

## Limitations

- A cartographical snapshot only restores the exact view when dragged back in with the same 2D/3D model and 'ili_input.csv loaded; changing either breaks reproducibility
- Maps dominated by a few high-intensity spots require manual hotspot-quantile reduction or Linear-to-Logarithmic scale switching to remain interpretable
- 'ili handles spatial data mapped either on an image or a 3D model — datasets without a matching model/background cannot be rendered in 3D

## Evidence

- [readme] README quote on 'ili purpose: [evidence span withheld — anchor status: external_doc]
