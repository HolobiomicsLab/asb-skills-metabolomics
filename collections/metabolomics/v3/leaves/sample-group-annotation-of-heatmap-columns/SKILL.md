---
name: sample-group-annotation-of-heatmap-columns
description: Use when input is an omics/gene expression matrix CSV (e.g. heatmap_test.csv)
  plus a sample-group annotation CSV (e.g. group_info.csv) and the goal is a publication-ready
  clustered heatmap with columns annotated by sample group.
when_to_use_negative:
- Input is a single unlabelled matrix with no sample grouping information — column
  annotation cannot be applied and a plain clustered heatmap suffices
- The goal is differential expression statistics or data preprocessing rather than
  visualization; GraphBio 'specifically focuses on facilitating the generation of
  publication-ready plots instead of data preprocessing and computing'
- The desired plot is another visualization type (volcano, MA, Venn, chord plot, etc.);
  GraphBio exposes these as separate modules with their own demo files
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: GraphBio
  role: Shiny web app whose heatmap module ingests heatmap_test.csv and group_info.csv
    uploaded by the user and produces the publication-ready annotated heatmap
  repo: https://github.com/databio2022/GraphBio
- name: pheatmap
  role: R package on which GraphBio's heatmap module is built; performs hierarchical
    clustering of rows/columns and group annotation of sample columns
- name: Shiny
  role: R framework used to implement GraphBio's web interface for uploading the expression
    matrix and group annotation file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fgene.2022.957317
    title: 'GraphBio: A shiny web app to easily perform popular visualization analysis
      for omics data'
  source_content_hash: sha256:6f4723fe0361adc555e342473c5e6382d9f9db1d84b04983f898666f93279a51
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Sample group annotation of heatmap columns

## Summary

Generate a clustered, group-annotated heatmap of a gene expression matrix using GraphBio's heatmap module (built on pheatmap), where sample columns are annotated with group labels supplied in a separate annotation CSV so the published figure shows both cluster structure and group membership.

## When to use

Use when input is an omics/gene expression matrix CSV (e.g. heatmap_test.csv) plus a sample-group annotation CSV (e.g. group_info.csv) and the goal is a publication-ready clustered heatmap with columns annotated by sample group.

## When NOT to use

- Input is a single unlabelled matrix with no sample grouping information — column annotation cannot be applied and a plain clustered heatmap suffices
- The goal is differential expression statistics or data preprocessing rather than visualization; GraphBio 'specifically focuses on facilitating the generation of publication-ready plots instead of data preprocessing and computing'
- The desired plot is another visualization type (volcano, MA, Venn, chord plot, etc.); GraphBio exposes these as separate modules with their own demo files

## Inputs

- Gene expression matrix CSV (heatmap_test.csv), samples as columns, features/genes as rows
- Sample group annotation CSV (group_info.csv), one group label per sample column
- Optional: user-prepared data formatted in Excel against the reference example files

## Outputs

- Clustered, group-annotated heatmap figure (e.g. heatmap.png) at publication-ready size and resolution
- Row/column hierarchical cluster structure visible alongside sample group annotation

## How to apply

Load the gene expression matrix (e.g. heatmap_test.csv, placed in graphbio-english/www/ of the GraphBio repository) and the sample annotation file group_info.csv, aligning each group label to the corresponding expression-matrix column. Render the heatmap with pheatmap, the package underlying GraphBio's heatmap module, enabling hierarchical clustering of both rows and columns and passing the group table as column annotation. On the GraphBio web app (http://www.graphbio1.com/en/) the user simply prepares both files in Excel and uploads them, since the module is designed for biologists without programming skills. Export the figure (e.g. as heatmap.png) at publication-ready size; GraphBio's defaults are tuned so 'the default figures are generally ideal for publication'. Decide between using the GraphBio web app directly or reproducing the render in R with pheatmap when scripted, reproducible output is needed.

## Related tools

- **GraphBio** (Shiny web app whose heatmap module ingests heatmap_test.csv and group_info.csv uploaded by the user and produces the publication-ready annotated heatmap) — https://github.com/databio2022/GraphBio
- **pheatmap** (R package on which GraphBio's heatmap module is built; performs hierarchical clustering of rows/columns and group annotation of sample columns)
- **Shiny** (R framework used to implement GraphBio's web interface for uploading the expression matrix and group annotation file)

## Examples

```
pheatmap(expr, annotation_col = group_info, cluster_rows = TRUE, cluster_cols = TRUE, filename = "heatmap.png")
```

## Evaluation signals

- The rendered figure reproduces the clustered, group-annotated heatmap style of GraphBio's heatmap module demo (Figure 1A)
- Every column of the expression matrix has a corresponding group label in group_info.csv (annotation table keys match column names exactly)
- Both row and column dendrograms (hierarchical clustering) are visible in the output figure
- Group annotation bar is rendered alongside the heatmap columns and matches group_info.csv values
- Exported figure resolution/size is adequate for publication (GraphBio defaults 'generally ideal for publication')

## Limitations

- No validation, benchmarking, or user evaluation of GraphBio is reported in the article
- No version number or license is stated in the article text (the Docker README tags v2.2.7-manual / v2.2.5-en-manual for self-deployment)
- Users must prepare data themselves in Excel following the reference example files; malformed or misaligned group_info.csv versus matrix columns will break annotation
- GraphBio's future roadmap notes additional visualization methods are not yet integrated, and the article omits formal benchmarking of figure quality

## Evidence

- [readme] README demo data listing: "Demo data for each visualization function (files are placed in the graphbio-english/www/ directory) 1. heatmap_test.csv and group_info.csv for heatmap."
- [intro] Excel preparation and default figure quality: "Users can easily prepare data to be visualized by Excel software based on given reference example files from GraphBio. The default figures are generally ideal for publication"
- [intro] pheatmap as the heatmap backend: "they were usually published as R packages, such as ggplot2 (Wickham, 2016), pheatmap for drawing heatmap"
- [abstract] Target audience and output claim: "It enables experimental biologists without programming skills to easily perform popular visualization analysis and get publication-ready figures"
