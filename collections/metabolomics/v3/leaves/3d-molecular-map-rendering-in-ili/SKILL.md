---
name: 3d-molecular-map-rendering-in-ili
description: Use when you have a feature quantification table from Optimus plus sample
  coordinates on a 3D model, and need to visualize where specific m/z features localize
  across anatomical regions.
when_to_use_negative:
- The samples are not spatially mapped — there are no coordinates linking LC-MS runs
  to positions on an image or 3D model
- The input is raw mzML from MassIVE — features must first be detected, aligned, quantified,
  and blank-filtered with Optimus before rendering in 'ili
- Direct-infusion runs with no spatial sampling design, which Optimus supports but
  which lack a map geometry to project onto
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3272
tools:
- name: '''ili'
  role: Web-application used to interactively visualize intensity maps of detected
    molecular features projected onto a 3D model; also loads and re-renders cartographical
    snapshots
  repo: https://github.com/ili-toolbox/ili
- name: Optimus
  role: Upstream KNIME/OpenMS workflow that generates the feature intensity table
    and spatial-mapping files consumed by 'ili (step 14)
  repo: https://github.com/MolecularCartography/Optimus
- name: OptimusViewer
  role: Complementary app to inspect XIC plots, MS peaks and MS2 spectra of the features
    being mapped before rendering
  repo: https://github.com/alexandrovteam/OptimusViewer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:718fd4032fb607a2f1b6f7021f5fb9c151191929060d4347968d427d27904044
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ili-toolbox/ili
  tools:
  - '''ili'
  - KNIME
  - Optimus
  - OptimusViewer
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# 3d-molecular-map-rendering-in-ili

## Summary

Renders spatial maps of LC-MS-detected molecular features onto a 3D model (e.g., a human body STL) using the 'ili web-application, completing the spatial-mapping stage of the Optimus molecular cartography protocol.

## When to use

Use when you have a feature quantification table from Optimus plus sample coordinates on a 3D model, and need to visualize where specific m/z features localize across anatomical regions.

## When NOT to use

- The samples are not spatially mapped — there are no coordinates linking LC-MS runs to positions on an image or 3D model
- The input is raw mzML from MassIVE — features must first be detected, aligned, quantified, and blank-filtered with Optimus before rendering in 'ili
- Direct-infusion runs with no spatial sampling design, which Optimus supports but which lack a map geometry to project onto

## Inputs

- Optimus spatial-mapping table (ili_input.csv) keyed by feature ID with per-sample intensities
- 3D model file (STL) of the sampled surface
- Sample coordinate/radius CSV mapping each LC-MS run to a position and spot size on the 3D model
- Experimental design file labeling runs (e.g., BLANK swabs vs samples)

## Outputs

- 3D molecular map rendered on the model surface for selected features
- Exported cartographical snapshots encoding the map and visualization settings
- Recorded spot patterns of each feature across anatomical regions

## How to apply

After the Optimus workflow produces the spatial-mapping table (ili_input.csv) keyed by feature ID with normalized, blank-filtered intensities, load it into 'ili together with the 3D model file (STL) and the sample coordinate/radius CSV. Select features of interest by m/z and retention time from the alignment output (e.g., m/z 355.215 for SLES, 225.075 for sinapic acid, 300.289 for C18-sphingosine) and render their intensity spots on the model surface. Record the spot pattern per anatomical region and export cartographical snapshots, which store the map and its visualization settings so the rendering can be shared and reloaded to reproduce the visualization exactly.

## Related tools

- **'ili** (Web-application used to interactively visualize intensity maps of detected molecular features projected onto a 3D model; also loads and re-renders cartographical snapshots) — https://github.com/ili-toolbox/ili
- **Optimus** (Upstream KNIME/OpenMS workflow that generates the feature intensity table and spatial-mapping files consumed by 'ili (step 14)) — https://github.com/MolecularCartography/Optimus
- **OptimusViewer** (Complementary app to inspect XIC plots, MS peaks and MS2 spectra of the features being mapped before rendering) — https://github.com/alexandrovteam/OptimusViewer

## Evaluation signals

- Each molecule's highest-intensity spots fall on the expected anatomical regions (e.g., sodium lauryl sulfate on scalp and hands, sinapic acid on hands, C18-sphingosine on the sole of the foot), matching the paper's reported distributions (Fig. 2b)
- The rendered feature IDs and m/z values in the map correspond to the Optimus alignment output (e.g., feature at m/z 300.289 for C18-sphingosine)
- Rendered snapshots are exported and reloading them reproduces the same visualization settings
- Blank swab runs are excluded from the map, consistent with the minimum 3.0 sample/blank intensity-ratio filter applied upstream

## Limitations

- 'ili visualization depends entirely on the quality of upstream Optimus output — features not filtered against blanks or normalized will produce misleading spot patterns
- The protocol is sized for molecular maps of about 100 spots and takes ~78 h overall (excluding reagent setup), so much larger spot counts are outside the demonstrated scope
- Optimus performs only putative annotation at Metabolomics Standards Initiative level 2 by m/z-RT matching; MS/MS validation of mapped annotations is not provided

## Evidence

- [intro] Creating spatial maps of detected features visualizable in 'ili app: "14. Creating spatial maps of detected features that can be visualized in [`ili app"
- [readme] Web-application for interactive visualization of spatial data mapped on an image or 3D model: "It is a web-application for interactive visualization of spatial data mapped either on an image or a 3D model, also developed by Alexandrov Team."
- [abstract] Cartographical snapshots are shareable, reloadable files reproducing map visualization settings: "we introduce cartographical snapshots: files that describe a particular map and visualization settings, and that can be shared and loaded to reproduce the visualization."
- [readme] 'ili is a visualization app for 3D molecular cartography: [evidence span withheld — anchor status: external_doc]
