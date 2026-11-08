---
name: cartographical-snapshot-sharing
description: Use when a spatial molecular map visualized in 'ili must be shared or
  re-displayed reproducibly, e.g. after tuning a logarithmic scale or hotspot quantile
  to reveal faint keypad traces, so the identical view can be restored.
when_to_use_negative:
- You only need a static publication figure and no reproducible interactive view —
  a PNG screenshot alone suffices.
- The molecular map and quantification matrix have not yet been produced by Optimus
  (ili_input.csv does not exist); run the Optimus workflow first.
- Data are plain LC-MS feature tables with no spatial coordinates; 'ili snapshots
  require spatially mapped data.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: '''ili'
  role: Web application in which the molecular map is visualized, display settings
    (logarithmic scale, hotspot quantile) are adjusted, and the cartographical snapshot
    is saved and reloaded
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: KNIME-based LC-MS workflow that upstream generates the spatial map (ili_input.csv)
    captured in the snapshot
  repo: https://github.com/MolecularCartography/Optimus
- name: OptimusViewer
  role: Companion application for inspecting XIC plots, MS peaks and MS2 spectra of
    features captured in the Optimus output underlying the map
  repo: https://github.com/alexandrovteam/OptimusViewer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:c2a0a61c210ff9dcc5d40d28665f1c913dab7ad50f4c206a0c02fdf859e70929
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - KNIME
  - Optimus
  - OptimusViewer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cartographical-snapshot-sharing

## Summary

Saving and sharing 'ili cartographical snapshots: files that record a molecular map together with its visualization settings so the exact 3D/2D view (e.g. the urocanate m/z 139.050 ATM keypad map) can be reloaded and reproduced by others.

## When to use

Use when a spatial molecular map visualized in 'ili must be shared or re-displayed reproducibly, e.g. after tuning a logarithmic scale or hotspot quantile to reveal faint keypad traces, so the identical view can be restored.

## When NOT to use

- You only need a static publication figure and no reproducible interactive view — a PNG screenshot alone suffices.
- The molecular map and quantification matrix have not yet been produced by Optimus (ili_input.csv does not exist); run the Optimus workflow first.
- Data are plain LC-MS feature tables with no spatial coordinates; 'ili snapshots require spatially mapped data.

## Inputs

- 'ili spatial map loaded from ili_input.csv (Optimus output) onto a 3D model in binary STL format
- Adjusted 'ili visualization state (scale mode, hotspot quantile) for the selected molecular feature map

## Outputs

- Cartographical snapshot file recording the map and its visualization settings
- PNG screenshot of the rendered molecular map (Ctrl/Cmd+S)

## How to apply

After loading the 3D model (binary STL) and ili_input.csv into 'ili and browsing to the molecular map of interest (e.g. urocanate at m/z 139.050), adjust display parameters such as logarithmic intensity scale or a reduced hotspot quantile so faint features (e.g. traces of buttons 8-0-1-7) become visible. Save the map and these visualization settings as a cartographical snapshot using Ctrl/Cmd+E; snapshots are shareable, reloadable files that reproduce a map's visualization settings. Separately export a PNG screenshot with Ctrl/Cmd+S for use as a static figure (e.g. Figure 2f). Sharing the snapshot lets collaborators reload the identical interactive view without re-running Optimus or re-tuning display options.

## Related tools

- **'ili** (Web application in which the molecular map is visualized, display settings (logarithmic scale, hotspot quantile) are adjusted, and the cartographical snapshot is saved and reloaded) — https://github.com/ili-toolbox/ili
- **Optimus** (KNIME-based LC-MS workflow that upstream generates the spatial map (ili_input.csv) captured in the snapshot) — https://github.com/MolecularCartography/Optimus
- **OptimusViewer** (Companion application for inspecting XIC plots, MS peaks and MS2 spectra of features captured in the Optimus output underlying the map) — https://github.com/alexandrovteam/OptimusViewer

## Evaluation signals

- Snapshot file exists and reloads in 'ili reproducing the same map, feature (e.g. m/z 139.050 urocanate) and display settings without manual re-tuning.
- PNG screenshot (Ctrl/Cmd+S) shows the intended visualization, e.g. high urocanate intensity on fingers and faint traces on keypad buttons 8-0-1-7.
- Reloading the snapshot on another machine yields the same visual appearance, confirming settings are fully encoded.
- Snapshot was saved only after display adjustments (log scale / reduced hotspot quantile) made faint spatial traces visible.

## Limitations

- Snapshots reproduce visualization only; regenerating the underlying map still requires re-running the Optimus workflow on the mzML data.
- The snapshot captures a chosen view — quantile or scale choices that hide or emphasize features are subjective and travel with the snapshot.
- Per the protocol paper's discussion section, 'No changelog found' for the associated repositories, so snapshot format/version compatibility is not explicitly documented.

## Evidence

- [readme] 'ili is a visualization app for 3D molecular cartography: [evidence span withheld — anchor status: external_doc]
- [other] Snapshot saving workflow from the ATM keypad task card: [evidence span withheld — anchor status: not_found]
- [other] Display adjustment needed to make faint traces visible before snapshotting: [evidence span withheld — anchor status: not_found]
- [other] Inputs to 'ili are the 3D model and ili_input.csv: [evidence span withheld — anchor status: not_found]
