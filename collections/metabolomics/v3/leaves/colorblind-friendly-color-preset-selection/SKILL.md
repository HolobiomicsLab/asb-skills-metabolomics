---
name: colorblind-friendly-color-preset-selection
description: Use when rendering omics visualization figures (heatmap, volcano, MA
  plot, etc.) and you need publication-ready default colors for readers with color
  vision deficiency instead of legacy tool palettes.
when_to_use_negative:
- You are doing data preprocessing or computation — GraphBio specifically focuses
  only on facilitating generation of publication-ready plots, not preprocessing
- A journal or figure requires a specific custom color scheme mandated by the publisher
  or co-authors, which overrides the default preset
- The plot type is not among GraphBio's 15 visualization methods (e.g., a bespoke
  multi-panel composite figure)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app whose modules ship colorblind-friendly default color presets
    for omics visualization figures
  repo: https://github.com/databio2022/GraphBio
- name: pheatmap
  role: R package underlying GraphBio's heatmap module; renders the clustered, group-annotated
    heatmap with the default color preset
- name: Shiny
  role: R framework in which GraphBio's visualization modules are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:5f4fa11b1fdc4f37a1c272d354b893ee9461fb07f409d4a376cc33783b9f386a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - pheatmap
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# colorblind-friendly-color-preset-selection

## Summary

Apply GraphBio's modern, colorblind-friendly default color presets when rendering omics plots such as pheatmap-based clustered heatmaps, so that figures are publication-ready without manual color tuning. This addresses the 'outdated color' limitation of older web-based omics visualization tools.

## When to use

Use when rendering omics visualization figures (heatmap, volcano, MA plot, etc.) and you need publication-ready default colors for readers with color vision deficiency instead of legacy tool palettes.

## When NOT to use

- You are doing data preprocessing or computation — GraphBio specifically focuses only on facilitating generation of publication-ready plots, not preprocessing
- A journal or figure requires a specific custom color scheme mandated by the publisher or co-authors, which overrides the default preset
- The plot type is not among GraphBio's 15 visualization methods (e.g., a bespoke multi-panel composite figure)

## Inputs

- Gene expression matrix CSV in GraphBio demo format (heatmap_test.csv)
- Sample/group annotation CSV with group labels per column (group_info.csv)

## Outputs

- Publication-ready figure file (e.g., heatmap.png) using the colorblind-friendly default color preset

## How to apply

Build the figure with the library the GraphBio module uses — the heatmap module is built on pheatmap — and keep the module's default color preset rather than specifying custom palettes. Prepare the data in Excel following GraphBio's reference example files (e.g., the gene expression matrix heatmap_test.csv plus the sample annotation file group_info.csv, with group labels aligned to expression matrix columns). Apply hierarchical clustering of rows and columns and sample group annotation, then export the figure (e.g., heatmap.png) at publication-ready size and resolution. Rationale: GraphBio was created because existing web tools suffer from 'outdated color', and its authors state 'the default figures are generally ideal for publication', so overriding the preset is usually unnecessary.

## Related tools

- **GraphBio** (Shiny web app whose modules ship colorblind-friendly default color presets for omics visualization figures) — https://github.com/databio2022/GraphBio
- **pheatmap** (R package underlying GraphBio's heatmap module; renders the clustered, group-annotated heatmap with the default color preset)
- **Shiny** (R framework in which GraphBio's visualization modules are implemented)

## Examples

```
pheatmap(expr, annotation_col = group_info, filename = "heatmap.png")
```

## Evaluation signals

- Rendered figure reproduces the clustered, group-annotated heatmap style of GraphBio's heatmap module demo (Figure 1A)
- Cluster structure of rows and columns is visible in the output heatmap
- Sample group annotation from group_info.csv is visible on the figure
- Figure is exported at publication-ready size and resolution (e.g., heatmap.png)
- Colors come from the module default preset — no manual palette override needed for publication readiness

## Limitations

- GraphBio reports no validation, benchmarking, or user evaluation of figure quality
- Existing web tools' limitation of 'outdated color' motivated the preset, but no explicit colorblind-assessment metric is given in the paper
- GraphBio focuses on plot generation, not data preprocessing or computing

## Evidence

- [intro] Default preset is publication-ready: "The default figures are generally ideal for publication"
- [intro] Motivation: outdated colors in prior web tools: "these tools come with limitations, such as many unnecessary parameters, single function, input formats, and outdated color"
- [readme] Heatmap demo inputs: "1. heatmap_test.csv and group_info.csv for heatmap."
- [intro] Excel-based data preparation workflow: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio"
