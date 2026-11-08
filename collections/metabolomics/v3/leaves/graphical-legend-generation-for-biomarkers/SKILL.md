---
name: graphical-legend-generation-for-biomarkers
description: Use when a GraPhlAn circular taxonomy tree highlights biomarker or background
  clades and a standalone legend image is needed to map annotations/colors back to
  clade names.
when_to_use_negative:
- Input is already an annotated GraPhlAn tree with a rendered legend — re-running
  export2graphlan would discard prior annotation choices
- Input is a plain phylogeny with no biomarker or metadata annotations (e.g., no LEfSe
  biomarkers and no --background_clades), so no legend content would be generated
- The goal is a heatmap rather than a circular tree — hclust2 handles clustered heat-maps
  with its own --legend_file mechanism instead
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN/BIOM output into the GraPhlAn tree and annotation
    files whose annotations drive the legend; annotation_legend_font_size sets legend
    typography
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated circular tree with matplotlib and emits the annotation-legend
    image (outimg_legend.png) alongside the main figure
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics backend used by GraPhlAn to draw the circular tree and its legends
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:456b95c76bd83f343b55dafeb96b60764526b54069d8ae9509ee9b72314fd941
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Graphical legend generation for biomarkers

## Summary

Generates publication-quality annotation and biomarker legends as side images during GraPhlAn circular-tree rendering. When export2graphlan converts MetaPhlAn/LEfSe/BIOM inputs to GraPhlAn tree and annotation files, the render step emits a legend image alongside the annotated tree figure so highlighted biomarker clades can be interpreted.

## When to use

Use when a GraPhlAn circular taxonomy tree highlights biomarker or background clades and a standalone legend image is needed to map annotations/colors back to clade names.

## When NOT to use

- Input is already an annotated GraPhlAn tree with a rendered legend — re-running export2graphlan would discard prior annotation choices
- Input is a plain phylogeny with no biomarker or metadata annotations (e.g., no LEfSe biomarkers and no --background_clades), so no legend content would be generated
- The goal is a heatmap rather than a circular tree — hclust2 handles clustered heat-maps with its own --legend_file mechanism instead

## Inputs

- GraPhlAn tree file (tree.txt) produced by export2graphlan.py from LEfSe-style or BIOM input
- GraPhlAn annotation file (annot.txt) produced by export2graphlan.py
- Annotated tree file produced by graphlan_annotate.py (outtree.txt)
- Legend font-size setting (annotation_legend_font_size, default 10) and clade-size/font-size defaults (10/20/200 and 10/8/12)

## Outputs

- Main circular taxonomic tree figure (outimg.png / outimg_annot.png, matplotlib-rendered PNG)
- Annotation-legend image (outimg_legend.png) mapping biomarker annotations to clade colors
- Annotated tree file (outtree.txt)

## How to apply

First run export2graphlan.py on the LEfSe-style or BIOM abundance table (e.g., -i saliva_microbiome BIOM) with defaults (def_clade_size 10, min_clade_size 20, max_clade_size 200, def_font_size 10, min_font_size 8, max_font_size 12, annotation_legend_font_size 10) to produce tree.txt and annot.txt; the annotation_legend_font_size parameter controls the legend typography. Attach the annotations to the tree with graphlan_annotate.py to obtain the annotated tree file (outtree.txt). Then render with GraPhlAn, which is matplotlib-backed, so that in addition to the main circular figure (outimg.png and outimg_annot.png) an annotation-legend image (outimg_legend.png) is emitted summarizing the biomarker/annotation colors. Validate that all expected outputs exist and that the legend image is non-empty and corresponds to the clades highlighted in the annotated figure.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN/BIOM output into the GraPhlAn tree and annotation files whose annotations drive the legend; annotation_legend_font_size sets legend typography) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated circular tree with matplotlib and emits the annotation-legend image (outimg_legend.png) alongside the main figure) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics backend used by GraPhlAn to draw the circular tree and its legends)

## Examples

```
export PATH=`pwd`/export2graphlan/:$PATH; export2graphlan.py -i saliva_microbiome.biom -t tree.txt -a annot.txt; graphlan_annotate.py tree.txt annot.txt; graphlan.py outtree.txt outimg_annot.png --annotations outimg_legend.png
```

## Evaluation signals

- All expected outputs exist: tree.txt, annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt
- outimg.png is a non-empty circular tree figure corresponding to the reference (e.g., Fig. S2 saliva microbiome)
- outimg_legend.png is non-empty and its entries match the clades annotated by export2graphlan (biomarker/background clades)
- The annotated tree (outtree.txt) was produced by graphlan_annotate.py from tree.txt + annot.txt, confirming the legend annotations are attached to the rendered tree
- Legend text is legible at the requested annotation_legend_font_size (default 10) within the min/max font-size bounds (8–12)

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM), so reproducing exact legend rendering may vary across versions
- export2graphlan requires pandas 0.13.1 and BIOM 2.0.1 (only for BIOM inputs) plus SciPy (required by hclust2), which may complicate environment setup
- Legend content depends on export2graphlan parameters such as abundance_threshold (default 20.0) and the clade-size bounds; mis-set parameters change which clades appear in the legend

## Evidence

- [other] export2graphlan generates the two GraPhlAn input files (tree file and annotation file), so the BIOM path can be exercised to render a circular taxonomic tree.: [evidence span withheld — anchor status: not_found]
- [readme] graphlan_annotate.py attaches derived annotations to the tree before final rendering: "The two output file of **export2graphlan** should then be used to run ``graphlan_annotate.py``, in order to attach to the tree the derived annotations, and finally, by executing ``graphlan.py`` the"
- [readme] Legend font size is an explicit export2graphlan parameter with default 10: "--annotation_legend_font_size ANNOTATION_LEGEND_FONT_SIZE
                        Set the font size for the annotation legend. Default is 10"
- [discussion] GraPhlAn rendering is matplotlib-backed: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] Annotated tree is an expected pipeline output: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
