---
name: root-histological-region-metabolite-mapping
description: Use when DESI-MSI data along a linear root developmental gradient have
  been clustered by DIMPLE and you need to identify which mass signatures are enriched
  in specific cell layers or in bimodal developmental zones.
when_to_use_negative:
- Input is raw, unclustered DESI-MSI spectra — DIMPLE must first process mass signatures
  along the linear gradient into clusters before histological inspection is meaningful
- Data are bulk metabolomics measurements with no spatial pixel information, so cell-layer
  localization cannot be assessed
- The tissue geometry is not a linear developmental axis (the DIMPLE design leverages
  'the linear developmental axis' of the root)
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: DESI-MSI
  role: Imaging modality producing the spatially resolved mass spectra (~50-100 µm
    resolution) that enable cell-layer-level metabolite localization in maize root
    sections
- name: DIMPLE
  role: Pipeline that processes mass signatures along the linear root gradient and
    generates hyperspectral channel clusters whose spatial patterns are inspected
    via the linescan GUI
  repo: github.com/dickinsonlab/DIMPLE-code
- name: METASPACE
  role: Open-source cloud platform used, including via the MSiReader v1.03 plug-in,
    to annotate unknown mass signatures from the enriched clusters
- name: Metlin Classic
  role: Metabolite search engine providing hits for m/z values to annotate cortex-
    and vasculature-enriched signatures
- name: LOTUS
  role: Open-source natural products identification tool used to annotate enriched
    mass signatures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:82ab427085d1e2b54d9846d192174426cabf686b660c0f0087f27d05d0701872
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - LOTUS
  - METASPACE
  - Metlin Classic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# root-histological-region-metabolite-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns DESI-MSI mass signatures to root histological regions (cortex vs. vasculature) and developmental zones (meristem vs. mature tissue) by inspecting DIMPLE hyperspectral channel clusters along the linear root axis, then annotating enriched m/z values with METASPACE, Metlin Classic, and LOTUS.

## When to use

Use when DESI-MSI data along a linear root developmental gradient have been clustered by DIMPLE and you need to identify which mass signatures are enriched in specific cell layers or in bimodal developmental zones.

## When NOT to use

- Input is raw, unclustered DESI-MSI spectra — DIMPLE must first process mass signatures along the linear gradient into clusters before histological inspection is meaningful
- Data are bulk metabolomics measurements with no spatial pixel information, so cell-layer localization cannot be assessed
- The tissue geometry is not a linear developmental axis (the DIMPLE design leverages 'the linear developmental axis' of the root)

## Inputs

- DIMPLE clustered hyperspectral channel outputs from DESI-MSI root scans (cdf files plus .mat Matlab workspaces, e.g., B73-root11, B73-root2-5, OG1)
- Root developmental gradient cluster assignments (e.g., Cluster 3 and Cluster 4) from DIMPLE processing
- Reference mass databases: METASPACE, Metlin Classic, LOTUS

## Outputs

- Table of bimodally enriched mass signatures (meristem + mature tissue), >=16 signatures expected
- Cell-layer-annotated signature table separating cortex-enriched (>15 signatures) from vasculature-enriched (>10 signatures) m/z values
- Metabolite annotations for enriched m/z values from METASPACE, Metlin Classic, and LOTUS searches

## How to apply

Load the DIMPLE clustered hyperspectral channel outputs (e.g., the cdf files and Matlab workspaces distributed with DIMPLE-code, such as B73-root11-workspace.mat) and extract channel assignments for the target clusters (e.g., Clusters 3-4 in B73 roots) along the root developmental gradient. Using the DIMPLE linescan GUI, identify mass signatures whose spatial signal is enriched at both the meristematic region and mature tissue (bimodal pattern) and record them in a signature table. For cell-layer assignment, inspect each channel's spatial localization within the root section to separate signatures enriched in the cortex from those enriched in the vasculature, since DESI-MSI's ~50-100 µm spatial resolution can distinguish these layers. Annotate the resulting cortex- and vasculature-enriched m/z values by searching METASPACE, Metlin Classic, and LOTUS. Compile the annotated tables and validate counts against reported minima (e.g., >=16 bimodal; >15 cortex-enriched; >10 vasculature-enriched) and against the corresponding supplemental figures/tables.

## Related tools

- **DESI-MSI** (Imaging modality producing the spatially resolved mass spectra (~50-100 µm resolution) that enable cell-layer-level metabolite localization in maize root sections)
- **DIMPLE** (Pipeline that processes mass signatures along the linear root gradient and generates hyperspectral channel clusters whose spatial patterns are inspected via the linescan GUI) — github.com/dickinsonlab/DIMPLE-code
- **METASPACE** (Open-source cloud platform used, including via the MSiReader v1.03 plug-in, to annotate unknown mass signatures from the enriched clusters)
- **Metlin Classic** (Metabolite search engine providing hits for m/z values to annotate cortex- and vasculature-enriched signatures)
- **LOTUS** (Open-source natural products identification tool used to annotate enriched mass signatures)

## Examples

```
load('B73-root11-workspace.mat'); % open DIMPLE Matlab workspace for B73 root 1 and inspect cluster channels in the linescan GUI
```

## Evaluation signals

- Bimodal signature table contains at least 16 mass signatures enriched at both meristematic region and mature tissue
- Cluster 4 cell-layer table contains more than 15 cortex-enriched and more than 10 vasculature-enriched mass signatures
- Compiled tables mirror the content of Supplemental Figures 5-7 and Supplemental Table 7
- Each enriched m/z value carries annotations cross-checked in METASPACE, Metlin Classic, and LOTUS
- Spatial assignments are consistent with the ~50-100 µm DESI-MSI pixel resolution relative to cortex/vasculature dimensions

## Limitations

- DESI-MSI spatial resolution (~50-100 µm) limits assignment to layers resolvable at that scale; finer cell-type distinctions (e.g., individual stem cell populations) may be masked
- The majority of metabolites in plants and animals remain unannotated, so many enriched m/z values may not receive database hits
- Workflow is tied to DIMPLE's assumption of a linear developmental axis; data not organized along such a gradient are outside scope

## Evidence
