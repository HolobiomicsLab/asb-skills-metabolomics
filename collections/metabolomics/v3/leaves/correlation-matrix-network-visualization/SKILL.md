---
name: correlation-matrix-network-visualization
description: Use when the input is a tabular correlation matrix (csv/txt/xls/xlsx)
  and the goal is a publication-ready network visualization of pairwise correlations
  without programming.
when_to_use_negative:
- Input is not a correlation matrix (e.g. a raw gene expression table for heatmap
  module, or DE results for volcano plots) — other GraphBio modules and demo files
  like heatmap_test.csv or volcano_example.csv apply
- Goal is computing correlations rather than visualizing an already-computed correlation
  matrix — GraphBio 'specifically focuses on facilitating the generation of publication-ready
  plots instead of data preprocessing and computing'
- User needs custom interactive network analysis beyond GraphBio's ggplot2-based default
  styling
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: GraphBio
  role: Shiny web application whose 'network plots' module consumes the correlation
    matrix and renders the network plot
  repo: https://github.com/databio2022/GraphBio
- name: Shiny
  role: R web framework underlying the GraphBio dashboard interface through which
    corr_net.csv is uploaded and the plot is rendered
  repo: https://github.com/databio2022/GraphBio
- name: ggplot2
  role: R plotting backend GraphBio uses to produce the network plot figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:71617366bfa542aad66b1cd7546d158366458d2dc4e27a013bf201a251abf257
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/databio2022/GraphBio
  tools:
  - GraphBio
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# correlation-matrix-network-visualization

## Summary

Render a network plot from a correlation matrix using GraphBio's 'network plots' Shiny module, whose demo/reference input is corr_net.csv, producing a publication-ready figure exportable in PDF, PNG, JPEG, or TIFF.

## When to use

Use when the input is a tabular correlation matrix (csv/txt/xls/xlsx) and the goal is a publication-ready network visualization of pairwise correlations without programming.

## When NOT to use

- Input is not a correlation matrix (e.g. a raw gene expression table for heatmap module, or DE results for volcano plots) — other GraphBio modules and demo files like heatmap_test.csv or volcano_example.csv apply
- Goal is computing correlations rather than visualizing an already-computed correlation matrix — GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing'
- User needs custom interactive network analysis beyond GraphBio's ggplot2-based default styling

## Inputs

- corr_net.csv — tabular correlation matrix file (GraphBio demo/reference input for the network plot module)
- user-supplied correlation matrix in csv, txt, xls, or xlsx format

## Outputs

- rendered network plot image with GraphBio's default publication-ready styling
- exported figure file in PDF, PNG, JPEG, or TIFF format with customizable size and resolution

## How to apply

Prepare the correlation matrix in Excel following GraphBio's reference example file corr_net.csv, which ships in the graphbio-english/www/ directory of the databio2022/GraphBio repository. Open the GraphBio Shiny web application (R Shiny dashboard built with bs4Dash) and select the 'network plots' module — one of its 15 visualization analysis methods. Upload the correlation matrix file (GraphBio accepts csv/txt/xls/xlsx) and render the plot with ggplot2 as the backend under the module's default parameters, which are 'generally ideal for publication'. Optionally adjust display parameters such as colors and node/label presentation while keeping the publication-ready defaults. Export the rendered figure in PDF, PNG, JPEG, or TIFF format with customizable size and resolution.

## Related tools

- **GraphBio** (Shiny web application whose 'network plots' module consumes the correlation matrix and renders the network plot) — https://github.com/databio2022/GraphBio
- **Shiny** (R web framework underlying the GraphBio dashboard interface through which corr_net.csv is uploaded and the plot is rendered) — https://github.com/databio2022/GraphBio
- **ggplot2** (R plotting backend GraphBio uses to produce the network plot figure)

## Examples

```
docker pull databio2022/graphbio:v2.2.5-en-manual; docker run --rm -d -p 80:3838 -v /root/log/:/home/shiny/graphbio-log/ databio2022/graphbio:v2.2.5-en-manual /init  # then open the web app and upload corr_net.csv to the 'network plots' module
```

## Evaluation signals

- Output figure must be a rendered network plot generated from corr_net.csv matching the module's default publication-ready styling
- The figure can be downloaded successfully in at least one of the supported export formats (PDF, PNG, JPEG, TIFF)
- Input file conforms to the structure of the reference demo file corr_net.csv (tabular, Excel-preparable)
- Deployed GraphBio instance serves the module — demo data files are placed in the graphbio-english/www/ directory

## Limitations

- No version number stated for GraphBio in the article (README pins docker tags v2.2.7-manual / v2.2.5-en-manual for Chinese/English versions)
- No validation, benchmarking, or user evaluation reported for the tool
- GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing' — upstream correlation computation is out of scope

## Evidence

- [readme] README lists corr_net.csv as the network plot demo input: "5. corr_net.csv for network plot"
- [readme] Demo data files location in the repository: "#### Demo data for each visualization function (files are placed in the graphbio-english/www/ directory)"
- [methods] Network plots are among GraphBio's 15 visualization modules: "GraphBio provides 15 visualization analysis modules, including heatmap, volcano plots, MA plots, network plots"
- [intro] Data preparation workflow via Excel and example files: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio."
- [intro] Default output styling is publication-ready: "The default figures are generally ideal for publication"
