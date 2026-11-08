---
name: omics-data-visualization
description: Use when a user has tabular omics data (e.g. expression matrix, DE results,
  survival, ROC inputs) already prepared in Excel against a reference example file
  and needs a publication-ready figure such as a heatmap, volcano plot, or chord plot.
when_to_use_negative:
- The goal is data preprocessing or heavy computation rather than plotting — GraphBio
  specifically focuses on facilitating the generation of publication-ready plots instead
  of data preprocessing and computing
- The user requires extensive custom plotting parameters — existing web tools' 'many
  unnecessary parameters' are exactly what GraphBio avoids, so only fine-tuning of
  defaults is supported
- The input data has not been shaped to match one of the module reference example
  files, since the app relies on users preparing data in Excel based on given reference
  example files
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: GraphBio
  role: The bs4Dash-based Shiny dashboard providing the 15 visualization analysis
    modules and example files that this skill operationalizes
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework on which GraphBio's module interface (run example, view example
    file, figure export) is built
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: Base R plotting package wrapped by GraphBio modules to render publication-ready
    figures with sensible defaults (R 4.0.3 environment)
  repo: https://github.com/databio2022/GraphBio
- name: pheatmap
  role: Wrapped package behind the heatmap module
- name: GOplot
  role: Wrapped package behind the chord plot module for GO analysis results
- name: FactoMineR
  role: Wrapped package for performing PCA analysis in the PCA module
- name: pROC
  role: Wrapped package for drawing ROC curves in the ROC analysis module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6356bf8f6228935e2c40c3d08cfbac1d582c5a4d17734d7a133c637bc4468e00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - FactoMineR
  - GOplot
  - GraphBio
  - Shiny
  - ggplot2
  - pROC
  - pheatmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# omics-data-visualization

## Summary

Generate publication-ready omics plots via GraphBio, a modular R Shiny (bs4Dash) dashboard that wraps established R visualization packages (ggplot2, pheatmap, GOplot, FactoMineR, factoextra, pROC, survminer) behind 15 standardized modules, so experimental biologists without programming skills can produce default figures that are ideal for publication.

## When to use

Use when a user has tabular omics data (e.g. expression matrix, DE results, survival, ROC inputs) already prepared in Excel against a reference example file and needs a publication-ready figure such as a heatmap, volcano plot, or chord plot.

## When NOT to use

- The goal is data preprocessing or heavy computation rather than plotting — GraphBio specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing
- The user requires extensive custom plotting parameters — existing web tools' 'many unnecessary parameters' are exactly what GraphBio avoids, so only fine-tuning of defaults is supported
- The input data has not been shaped to match one of the module reference example files, since the app relies on users preparing data in Excel based on given reference example files

## Inputs

- Tabular omics data files in csv, txt, xls, or xlsx format prepared in Excel against a module-specific reference example file (e.g. heatmap_test.csv with group_info.csv, volcano_example.csv, surv_example.csv, roc_example.csv, venn_example.csv)
- Demo data for each visualization function placed in the graphbio-english/www/ directory of the GraphBio repository

## Outputs

- Publication-ready figures with sensible defaults (e.g. heatmap, volcano plot, MA plot, network plot, dot plot, chord plot, pie plot, four quadrant diagram, Venn diagram, cumulative distribution curve, PCA, survival curve, ROC curve)
- Exported figure files with configurable size and resolution in PNG format

## How to apply

Select the matching module among GraphBio's 15 visualization analysis methods (heatmap, volcano plots, MA plots, network plots, dot plots, chord plots, pie plots, four quadrant diagrams, Venn diagrams, cumulative distribution curves, PCA, survival analysis, ROC analysis, correlation analysis, text cluster analysis). Prepare the input table in Excel following the module's reference example file; accepted formats are csv, txt, xls, and xlsx, where csv files are not UTF-8 encoded and txt files are tab-separated. Run the module via the bs4Dash-based Shiny dashboard, using the 'run example' and 'view example file' buttons to sanity-check the expected input shape, then upload user data. Keep default plotting parameters first, since the default figures are generally ideal for publication and only fine-tuning is needed in some cases; the plots are implemented in R (R 4.0.3 environment) by wrapping ggplot2 and the package corresponding to the analysis type (e.g. pheatmap for heatmaps, pROC for ROC, FactoMineR/factoextra for PCA, GOplot for chord plots). Export the rendered figure with configurable size and resolution in PNG format.

## Related tools

- **GraphBio** (The bs4Dash-based Shiny dashboard providing the 15 visualization analysis modules and example files that this skill operationalizes) — https://github.com/databio2022/GraphBio
- **Shiny** (R web framework on which GraphBio's module interface (run example, view example file, figure export) is built) — https://github.com/databio2022/GraphBio
- **ggplot2** (Base R plotting package wrapped by GraphBio modules to render publication-ready figures with sensible defaults (R 4.0.3 environment)) — https://github.com/databio2022/GraphBio
- **pheatmap** (Wrapped package behind the heatmap module)
- **GOplot** (Wrapped package behind the chord plot module for GO analysis results)
- **FactoMineR** (Wrapped package for performing PCA analysis in the PCA module)
- **pROC** (Wrapped package for drawing ROC curves in the ROC analysis module)

## Examples

```
docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init
```

## Evaluation signals

- The module script runs without error on a small self-made example input file following the reference example format
- The exported figure file exists and is in PNG format with configurable size and resolution
- The default styling of the output figure is judged as publication-ready against the paper's criterion that 'The default figures are generally ideal for publication'
- The input file conforms to one of the four accepted formats (csv not UTF-8 encoded, tab-separated txt, xls, xlsx) matching the chosen module's reference example
- The figure corresponds to one of the 15 documented visualization analysis methods and was produced through the dashboard workflow including the 'run example' check

## Limitations

- GraphBio is designed for visualization only — it does not perform data preprocessing or computing
- No version number or license is stated in the article sections; the Docker tags (e.g. v2.2.7-manual, v2.2.5-en-manual) are the version reference
- csv inputs are not UTF-8 encoded and txt inputs are tab-separated, so files exported in other encodings or delimiters may fail
- No validation, benchmarking, or user evaluation of the app is reported in the paper
- The tool targets experimental biologists without programming skills; users needing fully bespoke ggplot2 customization would need to work with the underlying R code directly

## Evidence

- [intro] Users prepare data in Excel against reference examples: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [abstract] No-programming audience and publication-ready output: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
- [intro] Scope excludes preprocessing: "GraphBio specifically focuses on facilitating the generation of"
- [methods] 15 available analysis modules: "GraphBio provides 15 visualization analysis modules, including heatmap, volcano plots, MA plots, network plots"
- [readme] Per-module demo data files: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [readme] Survival module example dataset: "13. surv_example.csv for survival curves"
- [intro] Shiny web framework foundation: "we developed an online web application called GraphBio using a shiny framework in R software"
