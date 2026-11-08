---
name: cross-modality-roi-assignment-handling
description: Use when joint protein-metabolite single-cell analysis in crowded human
  tissues requires correctly registered IMC and SIMS modalities per imaging region,
  e.g. before computing neighbor-based metabolite competition ratios or distance-to-vessel
  maps.
when_to_use_negative:
- The raw IMC and SIMS images have not yet been registered via template matching/affine
  transformation — computing spatial neighbor sets on unregistered data would misassign
  neighbors
- The TOF-SIMS dataset lacks the Phosphate 79 m/z channel, so the cross-modality registration
  anchors cannot be computed
- The analysis is purely intra-modal (e.g. IMC-only protein clustering) and does not
  require metabolite-protein spatial correspondence
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: scSpaMet
  role: Provides the notebook 02 IMC-SIMS registration/ROI assignment pipeline, the
    'spatial' folder of spatial interaction functions used to define neighborhoods,
    and 'utils.py' plotting helpers used to render competition-ratio spatial maps
  repo: https://github.com/coskunlab/scSpaMet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:2007139245ba3767a2a1b793d7ec32e2d6674a5550f537a653d3a2ccab35c544
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coskunlab/scSpaMet
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-modality-roi-assignment-handling

## Summary

Handles the registration of TOF-SIMS (metabolite) and IMC (protein) images across imaging regions of interest and carries the resulting per-cell ROI assignments into downstream single-cell analyses, so that cell-type-specific metabolite competition can be mapped as a function of distance to landmarks such as CD31+ endothelial cells.

## When to use

Use when joint protein-metabolite single-cell analysis in crowded human tissues requires correctly registered IMC and SIMS modalities per imaging region, e.g. before computing neighbor-based metabolite competition ratios or distance-to-vessel maps.

## When NOT to use

- The raw IMC and SIMS images have not yet been registered via template matching/affine transformation — computing spatial neighbor sets on unregistered data would misassign neighbors
- The TOF-SIMS dataset lacks the Phosphate 79 m/z channel, so the cross-modality registration anchors cannot be computed
- The analysis is purely intra-modal (e.g. IMC-only protein clustering) and does not require metabolite-protein spatial correspondence

## Inputs

- TOF-SIMS metabolite image stack (including the Phosphate 79 m/z channel) per imaging region
- IMC proteomic image stack (including Histone 3 and Intercalator channels) per imaging region
- Per-cell metabolite intensity table and cell centroid coordinates from single-cell extraction (notebook 04)
- Registered region-of-interest assignments from IMC-SIMS registration (notebook 02)

## Outputs

- Bi-modal matched (affine-transformed) IMC-SIMS images per imaging region
- Per-cell table with registered ROI assignments and spatial neighbor sets (20-um radius or cell-cell contact)
- Competition-ratio spatial maps, one per imaging region, for channels such as 148.0 m/z Methionine, 145.0 m/z Glutamine, 42.0 m/z lipid, 94.0 m/z, and 45 m/z lipid fragments

## How to apply

Register the two modalities per ROI using scSpaMet's cross-modality single-cell registration pipeline, which anchors on the Histone 3 and Intercalator markers in the IMC dataset and the Phosphate 79 m/z channel in the TOF-SIMS dataset; template matching locates the corresponding region, then the rotation offset is calculated and an affine transformation yields the bi-modal matched images (notebook 02). Load the registered ROI assignments together with the per-cell metabolite intensity table and centroids from notebook 04. Build spatial neighbor sets per cell using the custom spatial interaction functions in the 'spatial' folder (e.g. a 20-um radius around the centroid or direct cell-cell contact). Then compute per-channel competition ratios as a cell's own intensity divided by the mean neighbor intensity (notebook '06 Metabolite analysis in different regions') and render them back onto tissue coordinates with plotting helpers from 'utils.py'. The rationale is that any mis-registration between the SIMS metabolite map and the IMC protein segmentation would corrupt neighbor identity and distance-to-CD31+ cell calculations, invalidating the competition analysis.

## Related tools

- **scSpaMet** (Provides the notebook 02 IMC-SIMS registration/ROI assignment pipeline, the 'spatial' folder of spatial interaction functions used to define neighborhoods, and 'utils.py' plotting helpers used to render competition-ratio spatial maps) — https://github.com/coskunlab/scSpaMet

## Examples

```
jupyter nbconvert --to notebook --execute notebooks/"02 Registration of IMC and SIMS images for different regions".ipynb && jupyter nbconvert --to notebook --execute notebooks/"06 Metabolite analysis in different regions".ipynb
```

## Evaluation signals

- A competition-ratio spatial map file is produced for every region of interest in the registered ROI assignments
- All computed competition-ratio values are finite and strictly positive, indicating every cell had at least one measurable neighbor within the 20-um radius or contact set
- Registration succeeded if the template matching, rotation offset, and affine transformation steps yield bi-modal matched images for each imaging region without dropped ROIs
- Downstream competition findings are reproducible, e.g. 148.0 m/z Methionine, 42.0 m/z lipid, 94.0 m/z, and 45 m/z lipid fragments show higher expression in tumor cells than T-cells at further distance from CD31+ cells, while 145.0 m/z Glutamine is higher in tumor cells near CD31+ cells
- Per-cell protein and metabolite intensities agree with the published single-cell counts (e.g. 19507 lung cancer cells profiled) when re-running extraction on the registered ROIs

## Limitations

- Relatively small imaging regions from tissue samples due to the trade-off between imaging speed and spatial resolution in the TOF-SIMS, limiting the number of ROIs and neighbors available
- Metabolite annotation and coverage for lipids, amino acids, and metabolic pathways from TOF-SIMS remains challenging, complicating interpretation of unnamed channels like 94.0 m/z and 45 m/z
- OCT-embedded frozen tissues preserve metabolites better than FFPE but preserve tissue structure less well, affecting registration quality and the need to incorporate cryo-TOF-SIMS imaging

## Evidence

- [results] Cross-modality registration anchors: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS"
- [results] Registration algorithm: [evidence span withheld — anchor status: not_found]
- [abstract] Local metabolite competition rationale: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
- [readme] Repository organization of registration and spatial code: ""spatial" folder contains custom code for spatial interaction functions"
- [readme] Notebook for metabolite analysis in regions: "06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis)"
