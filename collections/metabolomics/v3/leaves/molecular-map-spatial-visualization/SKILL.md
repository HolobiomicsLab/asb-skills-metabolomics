---
name: molecular-map-spatial-visualization
description: Use when feature intensities with sample x/y/z coordinates and spot radii
  (e.g. Optimus 'ili_input.csv) must be overlaid on a 2D image or 3D STL model, and
  the resulting molecular map view should be tuned, saved, and reproducibly shared.
when_to_use_negative:
- The data are direct-infusion or non-spatial LC-MS runs with no x/y/z coordinates
  and no 2D/3D model to map onto.
- The input is a raw feature quantification matrix that first requires Optimus feature
  detection, alignment, filtering, and 'Save 'ili input file' node execution — visualization
  in 'ili is premature.
- Quantitative reprocessing (feature detection, blank filtering, normalization) is
  the goal; 'ili is visualization-only and does not reprocess mzML data.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: '''ili'
  role: Web application for interactive visualization of spatial molecular data mapped
    on an image or 3D model; handles spot rendering, hotspot quantile/log scaling,
    and cartographical snapshot capture via Ctrl/Cmd+E
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: Produces 'ili_input.csv from mzML-derived feature intensities, coordinates,
    and radii via the 'Read coordinates' and 'Save 'ili input file' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: OptimusViewer
  role: Complementary desktop app for inspecting XIC plots, MS peaks and MS2 spectra
    of detected features while curating which maps to visualize
  repo: https://github.com/alexandrovteam/OptimusViewer
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:275de5a5f09716bd4878e9cba727ca2ad39d6dec4e908c0addd49a1c0e3efda1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - KNIME
  - Optimus
  - OptimusViewer
  - Python
  - pyopenms
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# molecular-map-spatial-visualization

## Summary

Interactive 2D/3D visualization of spatially mapped LC-MS molecular features in the 'ili web application, including fine-tuning spot rendering, intensity scaling, and capturing shareable 'cartographical snapshots' (JSON files) that reproduce an exact map view.

## When to use

Use when feature intensities with sample x/y/z coordinates and spot radii (e.g. Optimus 'ili_input.csv) must be overlaid on a 2D image or 3D STL model, and the resulting molecular map view should be tuned, saved, and reproducibly shared.

## When NOT to use

- The data are direct-infusion or non-spatial LC-MS runs with no x/y/z coordinates and no 2D/3D model to map onto.
- The input is a raw feature quantification matrix that first requires Optimus feature detection, alignment, filtering, and 'Save 'ili input file' node execution — visualization in 'ili is premature.
- Quantitative reprocessing (feature detection, blank filtering, normalization) is the goal; 'ili is visualization-only and does not reprocess mzML data.

## Inputs

- 'ili_input.csv produced by Optimus (feature intensities, sample x/y/z coordinates, spot radii)
- Binary STL 3D model or PNG 2D background image
- 5-column coordinates file (Sample, x, y, z, radius) with no empty entries
- Previously saved cartographical_snapshot.json (for reproducibility validation)

## Outputs

- Interactive 2D/3D molecular map of a selected feature (e.g. urocanate m/z 139.050)
- cartographical_snapshot.json recording selected map, colormap, and view position
- PNG screenshot (Ctrl/Cmd+S)

## How to apply

Drag and drop a binary STL 3D model (or PNG 2D background) together with 'ili_input.csv into 'ili, then browse per-feature molecular maps and adjust the spot radii 'Size' factor in the 'Spots' menu so neighboring spots do not overlap. For maps dominated by a few high-intensity spots, reduce the 'hotspot quantile' value or switch the scale from 'Linear' to 'Logarithmic' so faint traces become visible. Fine-tune parameters in the '2D' or '3D' tab of the 'ili sidebar and navigate with scroll-to-zoom and left-drag-to-move. Capture the cartographical snapshot with Ctrl/Cmd+E, which writes a JSON file (selected map, colormap, view position) to the downloads directory; validate by dragging it back into 'ili with the same 2D/3D model and 'ili_input.csv loaded and confirming the identical view is restored.

## Related tools

- **'ili** (Web application for interactive visualization of spatial molecular data mapped on an image or 3D model; handles spot rendering, hotspot quantile/log scaling, and cartographical snapshot capture via Ctrl/Cmd+E) — https://github.com/ili-toolbox/ili
- **Optimus** (Produces 'ili_input.csv from mzML-derived feature intensities, coordinates, and radii via the 'Read coordinates' and 'Save 'ili input file' nodes) — https://github.com/MolecularCartography/Optimus
- **OptimusViewer** (Complementary desktop app for inspecting XIC plots, MS peaks and MS2 spectra of detected features while curating which maps to visualize) — https://github.com/alexandrovteam/OptimusViewer

## Examples

```
Drag-and-drop the binary STL model and ili_input.csv into the 'ili app (https://github.com/ili-toolbox/ili), then press Ctrl/Cmd+E to save cartographical_snapshot.json and Ctrl/Cmd+S for a PNG screenshot.
```

## Evaluation signals

- Dragging the saved cartographical_snapshot.json back into 'ili with the same 2D/3D model and 'ili_input.csv loaded restores exactly the same view (selected map, colormap, view position).
- The snapshot file is valid JSON and contains the visualization settings (selected map, colormap, view position).
- In the urocanate reference case, the map shows high intensity on the fingers and lower-intensity traces on the ATM keypad buttons matching the entered PIN 8-0-1-7.
- Neighboring spots do not visually overlap after adjusting the radii 'Size' factor in the 'Spots' menu.
- Rosemary validation: maps show rosmarinic acid enriched in old leaves, methoxycarnosic acid in young leaves, and cirsimaritin with a similar old-leaf-enriched distribution.

## Limitations

- The cartographical snapshot only restores the exact view when reloaded with the same 2D/3D model and 'ili_input.csv; changing either breaks reproduction.
- 'ili is a visualization app only — it does not perform LC-MS feature detection, alignment, filtering, or annotation, which must be done upstream in Optimus.
- For maps dominated by a few high-intensity spots, default linear scaling and default hotspot quantile can hide faint traces (e.g. keypad key signals) unless log scale or reduced quantile is applied.

## Evidence

- [full_text] Hotspot quantile and logarithmic scaling reveal faint traces: [evidence span withheld — anchor status: not_found]
- [full_text] Snapshot capture and reproduction workflow: [evidence span withheld — anchor status: not_found]
- [readme] 'ili input requires STL model and 'ili_input.csv with coordinates and radii: [evidence span withheld — anchor status: external_doc]
