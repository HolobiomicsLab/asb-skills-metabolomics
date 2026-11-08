---
name: metagenomic-taxonomy-tree-annotation
description: Use when you have a MetaPhlAn/LEfSe-style abundance table or a BIOM-format
  16S table (e.g., from QIIME or mothur, such as a saliva microbiome profile) and
  need a compact circular taxonomy tree highlighting biomarker clades.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair or a Newick phylogeny — export2graphlan
  is only for converting abundance/BIOM/LEfSe tables
- The goal is interactive exploration of large phylogenies rather than a static publication
  figure (the paper positions iTOL for interactive analyses and Krona for hierarchical
  quantitative browsing)
- The data are functional profiles not destined for clade-level taxonomy visualization
  and no GraPhlAn rendering is desired
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, HUMAnN, or BIOM abundance input into the GraPhlAn
    tree and annotation files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Annotates (graphlan_annotate.py) and renders (graphlan.py) the circular taxonomic
    tree using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to produce the output images
- name: hclust2
  role: Companion SegataLab tool for heatmap visualization of MetaPhlAn2 abundance
    profiles
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:97831d127e29ac1a02c1c364c1acf9c39507388f9be0a0eefe8a36845be358a0
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metagenomic-taxonomy-tree-annotation

## Summary

Converts metagenomic taxonomic abundance tables — including LEfSe-style and BIOM-format 16S profiles — into GraPhlAn tree and annotation files using export2graphlan, then annotates and renders them as publication-quality circular taxonomic trees with GraPhlAn.

## When to use

Use when you have a MetaPhlAn/LEfSe-style abundance table or a BIOM-format 16S table (e.g., from QIIME or mothur, such as a saliva microbiome profile) and need a compact circular taxonomy tree highlighting biomarker clades.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair or a Newick phylogeny — export2graphlan is only for converting abundance/BIOM/LEfSe tables
- The goal is interactive exploration of large phylogenies rather than a static publication figure (the paper positions iTOL for interactive analyses and Krona for hierarchical quantitative browsing)
- The data are functional profiles not destined for clade-level taxonomy visualization and no GraPhlAn rendering is desired

## Inputs

- LEfSe-style abundance table (e.g., MetaPhlAn output such as hmp_metahit.txt)
- BIOM-format 16S abundance table (e.g., saliva_microbiome.biom from QIIME or mothur pipelines)
- Optional LEfSe biomarker result file merged with the abundance table

## Outputs

- tree.txt — GraPhlAn tree file from export2graphlan
- annot.txt — GraPhlAn annotation file from export2graphlan
- outtree.txt — annotated tree produced by graphlan_annotate.py
- outimg.png — rendered circular taxonomic tree figure
- outimg_annot.png and outimg_legend.png — annotated figure and legend images

## How to apply

Add the export2graphlan directory to PATH (export PATH=`pwd`/export2graphlan/:$PATH) so export2graphlan.py is callable, then invoke it with the abundance/BIOM file via -i, requesting a tree file (-t tree.txt) and annotation file (-a annot.txt); defaults set def_clade_size 10, min_clade_size 20, max_clade_size 200, def_font_size 10, min_font_size 8, max_font_size 12, and annotation_legend_font_size 10. export2graphlan (ver. 0.2.1) converts MetaPhlAn, LEfSe, and/or HUMAnN output — and accepts BIOM input when biom-format ver. 2.0.1 is installed alongside pandas 0.13.1 and SciPy — sizing clades by LEfSe biomarker status and abundance threshold (default 20.0). Annotate the produced tree with graphlan_annotate.py to obtain the annotated tree (outtree.txt), then render with GraPhlAn, which is matplotlib-backed, to generate the circular figure plus annotation-legend and biomarker-legend images. BIOM input is exercised by passing the saliva microbiome BIOM file directly; --internal_levels can sum abundances from leaf to root on internal nodes when deeper-level representation is needed.

## Related tools

- **export2graphlan** (Converts MetaPhlAn, LEfSe, HUMAnN, or BIOM abundance input into the GraPhlAn tree and annotation files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Annotates (graphlan_annotate.py) and renders (graphlan.py) the circular taxonomic tree using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to produce the output images)
- **hclust2** (Companion SegataLab tool for heatmap visualization of MetaPhlAn2 abundance profiles) — https://github.com/SegataLab/hclust2

## Examples

```
export PATH=`pwd`/export2graphlan/:$PATH; export2graphlan.py -i saliva_microbiome.biom -t tree.txt -a annot.txt; graphlan_annotate.py tree.txt outtree.txt; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All expected outputs exist: tree.txt, annot.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a non-empty, matplotlib-rendered circular taxonomic tree corresponding to the reference figure (e.g., Fig. S2 for the saliva microbiome BIOM input)
- tree.txt and annot.txt contain the expected GraPhlAn tree structure and clade annotations derived from the input abundance/BIOM table
- Clade node sizes and font sizes fall within the configured ranges (min/max clade size 20–200, font size 8–12) with non-biomarker clades at the default size 10
- Running export2graphlan on a BIOM input completes without error, confirming the biom-format dependency is satisfied

## Limitations

- No software version numbers are stated for GraPhlAn itself or its dependencies beyond export2graphlan ver. 0.2.1 (27 October 2018), pandas 0.13.1, and BIOM 2.0.1, complicating reproducibility
- No changelog is found in the repository documentation, so behavioral changes between versions cannot be traced
- export2graphlan requires biom-format only when input files are in BIOM format; without it the BIOM path fails
- Annotation of clades depends on an abundance threshold (default 20.0) and LEfSe biomarker detection, so low-abundance clades may be dropped from the annotation

## Evidence

- [readme] export2graphlan requires biom-format only when input files are in BIOM format: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [readme] export2graphlan converts MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format: "export2graphlan.py (ver. 0.2.1 of 27 October 2018). Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format."
- [methods] Annotated tree is produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [methods] internal_levels sums abundances from leaf to root on internal nodes: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [discussion] GraPhlAn uses matplotlib as its rendering backend: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
