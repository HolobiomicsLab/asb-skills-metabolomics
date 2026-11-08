---
name: cartographical-snapshot-reproducibility
description: Use when a finalized 'ili molecular map view (spot rendering on an STL
  3D model or PNG 2D background) must be saved, shared, or exactly reproduced later,
  or when validating that a received cartographical_snapshot.json restores the intended
  view.
when_to_use_negative:
- When the goal is first-pass feature detection/quantification rather than view reproduction
  — that belongs to the Optimus workflow (mzML feature detection, alignment, filtering),
  before any snapshot exists.
- When the underlying 'ili_input.csv or the 2D/3D model has changed since the snapshot
  was taken — reload fidelity is only guaranteed for the same model and input table.
- When a different visualization app is used (e.g., OptimusViewer, which renders XIC
  plots, MS peaks and MS2 spectra from the Optimus *.db output rather than spatial
  map views).
tools:
- name: '''ili'
  role: Web-application in which the cartographical snapshot is captured (Ctrl/Cmd+E)
    and reloaded by drag-and-drop to reproduce the map view
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: Produces the 'ili_input.csv feature-intensity table consumed by the snapshot
    workflow
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:6f41cd89dcc8aa9cc85543be6e4b94441412d580359fec1722313bd2569e238b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - Optimus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cartographical-snapshot-reproducibility

## Summary

Use 'ili's cartographical snapshots (Ctrl/Cmd+E) to save a molecular map view as a JSON file that records the selected map, colormap, and view position, so the exact visualization can be shared and reproduced by reloading the snapshot with the same 2D/3D model and 'ili_input.csv.

## When to use

Use when a finalized 'ili molecular map view (spot rendering on an STL 3D model or PNG 2D background) must be saved, shared, or exactly reproduced later, or when validating that a received cartographical_snapshot.json restores the intended view.

## When NOT to use

- When the goal is first-pass feature detection/quantification rather than view reproduction — that belongs to the Optimus workflow (mzML feature detection, alignment, filtering), before any snapshot exists.
- When the underlying 'ili_input.csv or the 2D/3D model has changed since the snapshot was taken — reload fidelity is only guaranteed for the same model and input table.
- When a different visualization app is used (e.g., OptimusViewer, which renders XIC plots, MS peaks and MS2 spectra from the Optimus *.db output rather than spatial map views).

## Inputs

- 'ili_input.csv feature-intensity table produced from Optimus mzML-derived output
- Binary STL 3D model or PNG 2D background image
- 5-column coordinates file (Sample, x, y, z, radius)
- Cartographical snapshot JSON file (cartographical_snapshot.json) when reproducing a saved view

## Outputs

- cartographical_snapshot.json storing selected map, colormap, spot/visualization settings, and view position
- Restored 'ili visualization identical to the saved map view (map, colormap, camera position)

## How to apply

Load the Optimus-derived 'ili_input.csv (feature intensities, sample x/y/z coordinates, spot radii) into 'ili together with the binary STL 3D model (or PNG 2D background) and the 5-column coordinates file (Sample, x, y, z, radius). Tune the view: set the spot 'Size' radii factor in the 'Spots' menu to prevent overlap, reduce the 'hotspot quantile' or switch scale from 'Linear' to 'Logarithmic' for hotspot-dominated maps, then navigate the 2D/3D sidebar view (scroll to zoom, left-drag to pan). Capture the snapshot with Ctrl/Cmd+E, which writes a JSON file with all visualization settings (selected map, colormap, view position) to the downloads directory; keep or rename it as cartographical_snapshot.json. Validation: drag and drop the JSON back into 'ili with the same 2D/3D model and 'ili_input.csv loaded, and confirm the exact view — selected map, colormap, and camera position — is restored, verifying the file parses as valid JSON containing those settings.

## Related tools

- **'ili** (Web-application in which the cartographical snapshot is captured (Ctrl/Cmd+E) and reloaded by drag-and-drop to reproduce the map view) — https://github.com/ili-toolbox/ili
- **Optimus** (Produces the 'ili_input.csv feature-intensity table consumed by the snapshot workflow) — https://github.com/MolecularCartography/Optimus

## Evaluation signals

- Dragging the saved JSON into 'ili with the same 2D/3D model and 'ili_input.csv restores exactly the same selected map, colormap, and view position.
- The snapshot file parses as valid JSON and contains entries for the selected map, colormap, and view position.
- The snapshot was written to the downloads directory upon the Ctrl/Cmd+E capture and is retained/renamed as cartographical_snapshot.json.
- Visualization decisions preceding the snapshot are recorded: spot radii 'Size' set so neighboring spots do not overlap, and hotspot-dominated maps handled via a reduced 'hotspot quantile' or Logarithmic scale.

## Limitations

- Snapshot restore requires the identical 2D/3D model and the same 'ili_input.csv; changing either breaks exact reproduction.
- The snapshot captures only visualization settings — the underlying molecular data must already have been generated by Optimus.
- Snapshot handling depends on downloads-directory file management and manual renaming to cartographical_snapshot.json.

## Evidence

- [abstract] Cartographical snapshots are introduced as shareable, reloadable files that reproduce a map's visualization settings: "we introduce cartographical snapshots: files that describe a particular map and visualization settings, and that can be shared and loaded to reproduce the visualization."
- [intro] Creating spatial maps of detected features visualizable in 'ili app: "14. Creating spatial maps of detected features that can be visualized in [`ili app]"
- [readme] README documents cartographical snapshots as a first-class 'ili capability: [evidence span withheld — anchor status: external_doc]
- [readme] README places snapshots in the context of 3D molecular cartography visualization: [evidence span withheld — anchor status: external_doc]
