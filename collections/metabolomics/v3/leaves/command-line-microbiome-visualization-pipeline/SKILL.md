---
name: command-line-microbiome-visualization-pipeline
description: Use when you need to convert BIOM-format 16S abundance tables or LEfSe-style
  outputs into publication-quality circular taxonomic trees using export2graphlan
  and GraPhlAn.
when_to_use_negative:
- Input is already a rendered GraPhlAn tree/annotation pair — the conversion step
  is unnecessary.
- The goal is interactive exploration of large phylogenies or hierarchical quantitative
  metagenomic profiles, where iTOL or Krona are the article's named alternatives.
- Input is not a taxonomic abundance/profile format convertible by export2graphlan
  (e.g., raw reads or non-hierarchical feature tables without taxonomic lineages).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: export2graphlan
  role: Converts BIOM/LEfSe/MetaPhlAn/HUMAnN output into GraPhlAn tree and annotation
    files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated circular taxonomic tree using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn to generate the circular tree
    images
- name: hclust2
  role: Companion heatmap tool from the same lab for clustered abundance profiles
    (optional in the visualization pipeline)
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:e2d56779252cb10cfa3d318b69c05a00afa44de0ab36317c405c1ffd027f2053
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# command-line-microbiome-visualization-pipeline

## Summary

A command-line pipeline that converts MetaPhlAn, LEfSe, HUMAnN, or BIOM-format abundance tables into GraPhlAn input files (tree + annotation) via export2graphlan, then renders compact circular taxonomic trees with the matplotlib-backed GraPhlAn tool.

## When to use

Use when you need to convert BIOM-format 16S abundance tables or LEfSe-style outputs into publication-quality circular taxonomic trees using export2graphlan and GraPhlAn.

## When NOT to use

- Input is already a rendered GraPhlAn tree/annotation pair — the conversion step is unnecessary.
- The goal is interactive exploration of large phylogenies or hierarchical quantitative metagenomic profiles, where iTOL or Krona are the article's named alternatives.
- Input is not a taxonomic abundance/profile format convertible by export2graphlan (e.g., raw reads or non-hierarchical feature tables without taxonomic lineages).

## Inputs

- BIOM-format 16S abundance table (e.g., from QIIME or mothur pipelines, such as a saliva microbiome dataset)
- LEfSe input file (taxonomic abundance table suitable for LEfSe biomarker analysis)
- MetaPhlAn2 or HUMAnN output abundance profiles

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (circular taxonomic tree figure)
- outimg_annot.png and outimg_legend.png (annotated image and legend images)

## How to apply

First put the export2graphlan directory on PATH (`export PATH=`pwd`/export2graphlan/:$PATH`) so the scripts are callable. Invoke export2graphlan.py with the BIOM or LEfSe-style abundance file via -i, requesting a tree file (-t) and annotation file (-a); defaults are def_clade_size 10, min_clade_size 20, max_clade_size 200, def_font_size 10, min_font_size 8, max_font_size 12, and annotation_legend_font_size 10. If BIOM input is used, the biom-format library (ver. 2.0.1) must be installed alongside pandas 0.13.1 and SciPy. Next, annotate the produced tree with graphlan_annotate.py to obtain the annotated tree, and render it with graphlan.py (matplotlib-backed) to produce the final circular image plus annotation-legend and biomarker-legend images. Validate by confirming the expected outputs (tree.txt, annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt) exist and that outimg.png is a non-empty circular tree figure.

## Related tools

- **export2graphlan** (Converts BIOM/LEfSe/MetaPhlAn/HUMAnN output into GraPhlAn tree and annotation files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated circular taxonomic tree using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn to generate the circular tree images)
- **hclust2** (Companion heatmap tool from the same lab for clustered abundance profiles (optional in the visualization pipeline)) — https://github.com/SegataLab/hclust2

## Examples

```
export PATH=`pwd`/export2graphlan/:$PATH; export2graphlan.py -i saliva_microbiome.biom -t tree.txt -a annot.txt; graphlan_annotate.py tree.txt annot.txt --outdir out; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All expected output files exist: tree.txt, annot.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png.
- outimg.png is a non-empty PNG depicting a circular taxonomic tree consistent with the reference figure (e.g., Fig. S2 for the saliva microbiome test case).
- export2graphlan.py exits without error when given a BIOM input, demonstrating the BIOM path is exercised end-to-end.
- The annotated tree (outtree.txt) produced by graphlan_annotate.py contains the derived annotations from annot.txt attached to the tree structure.
- Rendered image legends (annotation-legend and biomarker-legend files) are produced alongside the main figure.

## Limitations

- export2graphlan requires pandas 0.13.1, BIOM 2.0.1 (only for BIOM inputs), and SciPy; missing these breaks the pipeline.
- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies in the article, so exact reproducibility of versions is not documented.
- Most available tools (including this pipeline's targets) are infeasible for displaying phylogenies with thousands of microbial taxa in hundreds of samples without careful clade-size and font-size parameterization.

## Evidence

- [other] export2graphlan generates the two GraPhlAn input files (tree file and annotation file), so the BIOM path can be exercised to render a circular taxonomic tree.: [evidence span withheld — anchor status: not_found]
- [readme] export2graphlan converts to GraPhlAn inputs: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn"
- [readme] BIOM library requirement for BIOM-format inputs: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [discussion] GraPhlAn depends on matplotlib for rendering: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] Default clade and font size parameters documented in README: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
