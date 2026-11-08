---
name: maize-root-spatial-metabolomics
description: Use when you have DESI-MSI hyperspectral images of maize (or other linearly
  organized) tissue sections and need to identify mass signatures enriched at specific
  positions along a developmental gradient or in specific cell layers, and to annotate
  unknown m/z values.
when_to_use_negative:
- The data are bulk metabolomics feature tables rather than spatially resolved MSI
  pixels — the skill's core value is spatial localization along the developmental
  axis
- The tissue has no linear organizational axis to exploit; DIMPLE is specifically
  designed to 'leverage the linear developmental axis'
- The goal is high spatial resolution below DESI-MSI's ∼50-100 µm limit, where matrix-assisted
  approaches like MALDI or SIMS would be more appropriate
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: DESI-MSI
  role: Acquires in situ mass spectrometry images of the developing maize root with
    ∼50-100 µm spatial resolution along the developmental gradient
- name: DIMPLE
  role: Processes mass signatures along the linear root axis and generates clusters
    of metabolites with specific developmental enrichment patterns
  repo: github.com/dickinsonlab/DIMPLE-code
- name: MSiReader v1.03
  role: MSI data analysis and visualization, including the METASPACE plug-in for annotating
    unknown mass signatures
- name: METASPACE
  role: Open-source cloud platform for spatial metabolite annotation of unknown m/z
    values
- name: Metlin Classic
  role: Metabolite search engine providing hits for m/z values during signature annotation
- name: LOTUS
  role: Open-source natural products database used to annotate mass signatures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:dd7769f42bb052580b733d550591722185e25d69411609261fe19f17884e4909
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
  - MSiReader v1.03
  - Metlin Classic
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# maize-root-spatial-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps and annotates metabolites along the linear maize root developmental axis using DESI-MSI (∼50-100 µm spatial resolution) and the DIMPLE pipeline, then curates cluster-specific mass signature tables and annotates m/z values with METASPACE, Metlin Classic, and LOTUS.

## When to use

Use when you have DESI-MSI hyperspectral images of maize (or other linearly organized) tissue sections and need to identify mass signatures enriched at specific positions along a developmental gradient or in specific cell layers, and to annotate unknown m/z values.

## When NOT to use

- The data are bulk metabolomics feature tables rather than spatially resolved MSI pixels — the skill's core value is spatial localization along the developmental axis
- The tissue has no linear organizational axis to exploit; DIMPLE is specifically designed to 'leverage the linear developmental axis'
- The goal is high spatial resolution below DESI-MSI's ∼50-100 µm limit, where matrix-assisted approaches like MALDI or SIMS would be more appropriate

## Inputs

- DESI-MSI hyperspectral imaging data of maize root sections (cdf files, e.g. B73-root11, B73-root2-5, B73-root-3-3 for B73; OG1, OG5, OG2-P11-0 for Oaxacan Green)
- Pre-computed Matlab workspaces (e.g. B73-root11-workspace.mat, OG1-workspace-2.mat)
- DIMPLE clustered hyperspectral channel outputs along the root developmental gradient

## Outputs

- Table of bimodal mass signatures enriched at both meristem and mature tissue (>=16 signatures)
- Cell-layer-annotated table of Cluster 4 cortex-enriched (>15) and vasculature-enriched (>10) m/z signatures
- Metabolite annotations for m/z values from METASPACE, Metlin Classic, and LOTUS searches
- Cluster assignments of mass signatures along the root developmental gradient (mirroring Supplemental Figures 5-7 and Supplemental Table 7)

## How to apply

Acquire DESI-MSI data from maize root sections (B73 salt-susceptible and Oaxacan Green salt-resilient varieties; cdf files with pre-computed Matlab workspaces are available, e.g. B73-root11 and B73-root11-workspace.mat for B73 root 1). Load the hyperspectral channel outputs into DIMPLE (Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation), which processes mass signatures along the linear root developmental axis and generates clusters with specific developmental enrichment patterns; batchcdfread (developed by Yifan Meng, Zare lab, Stanford) handles reading the cdf files. Extract channel assignments for clusters of interest along the root gradient (e.g. Clusters 3-4 in B73 roots), then inspect each cluster in the linescan GUI to identify bimodal signatures enriched at both the meristematic region and mature tissue, and cell-layer-specific signatures such as cortex- versus vasculature-enriched masses in Cluster 4. Annotate the extracted m/z values using METASPACE (MSiReader v1.03 plug-in), Metlin Classic, and LOTUS. Compile signature tables mirroring Supplemental Figures 5-7 and Supplemental Table 7, meeting the reported minimum counts (>=16 bimodal; >15 cortex-enriched; >10 vasculature-enriched signatures).

## Related tools

- **DESI-MSI** (Acquires in situ mass spectrometry images of the developing maize root with ∼50-100 µm spatial resolution along the developmental gradient)
- **DIMPLE** (Processes mass signatures along the linear root axis and generates clusters of metabolites with specific developmental enrichment patterns) — github.com/dickinsonlab/DIMPLE-code
- **MSiReader v1.03** (MSI data analysis and visualization, including the METASPACE plug-in for annotating unknown mass signatures)
- **METASPACE** (Open-source cloud platform for spatial metabolite annotation of unknown m/z values)
- **Metlin Classic** (Metabolite search engine providing hits for m/z values during signature annotation)
- **LOTUS** (Open-source natural products database used to annotate mass signatures)

## Examples

```
load('B73-root11-workspace.mat'); channels = batchcdfread('B73-root11'); % inspect Cluster 3-4 linescan channels in the DIMPLE GUI for bimodal and cortex/vasculature-enriched m/z
```

## Evaluation signals

- Bimodal signature table contains at least 16 mass signatures enriched at both meristematic region and mature tissue
- Cluster 4 cortex-enriched signatures exceed 15 and vasculature-enriched signatures exceed 10 (matching Supplemental Figures 6-7 and Supplemental Table 7)
- Compiled tables mirror Supplemental Figures 5-7 and Supplemental Table 7 in content
- Cluster assignments reproduce the developmental gradient channel structure for Clusters 3-4 in B73 roots
- Each extracted m/z value receives annotation hits from at least one of METASPACE, Metlin Classic, or LOTUS

## Limitations

- DESI-MSI spatial resolution is only ∼50-100 µm, limiting separation of rare cell types such as stem cells whose signatures bulk-scale analyses mask
- The majority of metabolites in plants and animals remain unannotated, so many extracted mass signatures may lack confident identifications even after METASPACE/Metlin/LOTUS searching
- Untargeted analysis of MSI datasets is data-heavy: each pixel contains a full mass spectrum with hundreds to thousands of peaks

## Evidence

- [other] cluster inspection and curation: [evidence span withheld — anchor status: not_found]
