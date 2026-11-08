---
name: omics-example-data-preparation
description: Use when a table must be shaped for one of GraphBio's 15 visualization
  modules (heatmap, chord plot, volcano, ROC, Venn, etc.) — locate the matching demo
  file and reformat the user's data in Excel to match it.
when_to_use_negative:
- The task is data preprocessing or statistical computing rather than plotting — GraphBio
  specifically focuses on facilitating generation of publication-ready plots, not
  preprocessing
- The target plot type is not among GraphBio's 15 integrated modules (heatmap, volcano,
  MA, network, dot, chord, pie, four quadrant, Venn, cumulative distribution, PCA,
  survival, ROC, correlation, text cluster)
- The user requires non-default styling or parameters beyond what the GraphBio module
  exposes
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GraphBio
  role: Shiny web app whose modules ship the per-function demo data files that serve
    as format templates for preparing user input
  repo: https://github.com/databio2022/GraphBio
- name: GOplot
  role: R backend wrapped by GraphBio's chord plot module; determines the gene-to-GO-term
    table layout that chord_example.csv demonstrates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:dac0ecfe234b289b6a408e49e13be9aa94d6f4cdbb7ebf4dc9423479a1d431a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GOplot
  - GraphBio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# omics-example-data-preparation

## Summary

Prepare user omics data for a GraphBio visualization module by mirroring the module's bundled reference example file (e.g. chord_example.csv for the GOplot-backed chord plot, volcano_example.csv, surv_example.csv) so the uploaded table matches the expected columns and renders a publication-ready figure with defaults.

## When to use

Use when a table must be shaped for one of GraphBio's 15 visualization modules (heatmap, chord plot, volcano, ROC, Venn, etc.) — locate the matching demo file and reformat the user's data in Excel to match it.

## When NOT to use

- The task is data preprocessing or statistical computing rather than plotting — GraphBio specifically focuses on facilitating generation of publication-ready plots, not preprocessing
- The target plot type is not among GraphBio's 15 integrated modules (heatmap, volcano, MA, network, dot, chord, pie, four quadrant, Venn, cumulative distribution, PCA, survival, ROC, correlation, text cluster)
- The user requires non-default styling or parameters beyond what the GraphBio module exposes

## Inputs

- User's omics result table (CSV prepared in Excel, e.g. GO analysis results linking genes to GO terms)
- Reference example file for the chosen module from graphbio-english/www/ (e.g. chord_example.csv, volcano_example.csv, ma_example.csv, roc_example.csv)

## Outputs

- Module-conformant input CSV ready for upload to GraphBio
- Publication-ready plot rendered by GraphBio (default export as PDF, e.g. chord_plot.pdf)

## How to apply

Identify the target GraphBio module and retrieve its reference demo file from the graphbio-english/www/ directory of the databio2022/GraphBio repository (e.g. chord_example.csv for chord plot, heatmap_test.csv plus group_info.csv for heatmap, roc_example.csv for ROC, venn_example.csv for Venn). Open the demo file in Excel to learn the required column layout and data types for that module. Reformat the user's own table in Excel to mirror the demo structure — for the chord plot module this is a GO analysis result table linking genes to GO terms, consumed by the GOplot R package backend (Walter et al., 2015). Upload the reformatted CSV through the GraphBio web app (http://www.graphbio1.com/en/ or the Docker image) and rely on the module's default parameters, which GraphBio sets so 'the default figures are generally ideal for publication'. Verify the output (e.g. chord_plot.pdf) renders the expected plot type with ribbons/elements corresponding to the input rows.

## Related tools

- **GraphBio** (Shiny web app whose modules ship the per-function demo data files that serve as format templates for preparing user input) — https://github.com/databio2022/GraphBio
- **GOplot** (R backend wrapped by GraphBio's chord plot module; determines the gene-to-GO-term table layout that chord_example.csv demonstrates)

## Examples

```
docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- Reformatted CSV column names and column order match the module's bundled example file exactly (e.g. chord_example.csv for the chord plot module)
- Each of the 15 modules has a documented demo file and the user file maps 1:1 to one of them per the README list
- Uploaded file produces a rendered plot without module errors and with GraphBio's default settings
- Exported figure (e.g. chord_plot.pdf) shows the expected plot type whose elements correspond to input rows (GO term–gene ribbons matching chord_example.csv entries)
- Figure is judged publication-ready without manual parameter tuning, per GraphBio's design goal

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio's workflow is reported in the article
- Additional visualization methods are not yet integrated; GraphBio plans to continue adding more, so some plot types lack demo files
- The English-version and Chinese-version Docker images carry different tags (v2.2.5-en-manual vs v2.2.7-manual), so the available module set may differ between deployments

## Evidence

- [intro] Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio"
- [readme] Demo data for each visualization function (files are placed in the graphbio-english/www/ directory): "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [methods] 13. surv_example.csv for survival curves: "13. surv_example.csv for survival curves"
- [intro] The default figures are generally ideal for publication: "The default figures are generally ideal for publication"
