---
name: metagenomic-abundance-table-conversion
description: Use when you have a taxonomy- or function-annotated abundance table (MetaPhlAn/LEfSe/HUMAnN
  text output or BIOM v1/v2 file, e.g. from QIIME or mothur 16S studies) and need
  GraPhlAn input files for circular visualization.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair or raw phylogenetic tree without
  abundance data — export2graphlan is a converter for abundance/biomarker tables.
- The goal is interactive exploration rather than a publication-quality static circular
  figure — tools like iTOL or Krona target that use case.
- The abundance table is unclassified at any taxonomic level and --internal_levels
  cannot reconcile hierarchy, so leaf-to-root summation is undefined.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts the abundance/biomarker table (text or BIOM) into the GraPhlAn tree.txt
    and annot.txt files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Annotates the tree via graphlan_annotate.py and renders the circular visualization
    using matplotlib
- name: hclust2
  role: Companion tool for heatmap plotting of MetaPhlAn2 abundance profiles in the
    same pipeline context
  repo: https://github.com/SegataLab/hclust2
- name: matplotlib
  role: Underlying rendering library used by GraPhlAn to draw the output images
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:24a6d3e11e7d8f0a01ec97507e1a00babbbcf5b7810e6fda282e257f42d42eef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metagenomic-abundance-table-conversion

## Summary

Convert metagenomic/metataxonomic abundance tables — including MetaPhlAn, LEfSe, HUMAnN outputs and BIOM-format files (e.g. 16S rRNA profiles such as vaginal microbiota data) — into the GraPhlAn tree.txt and annot.txt files using export2graphlan, for rendering circular phylogenetic visualizations.

## When to use

Use when you have a taxonomy- or function-annotated abundance table (MetaPhlAn/LEfSe/HUMAnN text output or BIOM v1/v2 file, e.g. from QIIME or mothur 16S studies) and need GraPhlAn input files for circular visualization.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair or raw phylogenetic tree without abundance data — export2graphlan is a converter for abundance/biomarker tables.
- The goal is interactive exploration rather than a publication-quality static circular figure — tools like iTOL or Krona target that use case.
- The abundance table is unclassified at any taxonomic level and --internal_levels cannot reconcile hierarchy, so leaf-to-root summation is undefined.

## Inputs

- MetaPhlAn abundance profile (e.g. hmp_metahit.txt)
- LEfSe biomarker result file
- HUMAnN functional profile output (KEGG ortholog groups)
- BIOM-format abundance table (versions 1 and 2), e.g. from QIIME or mothur 16S rRNA studies

## Outputs

- tree.txt — GraPhlAn tree file
- annot.txt — GraPhlAn annotation file
- outtree.txt — annotated tree from graphlan_annotate.py
- outimg.png — rendered circular taxonomy tree
- outimg_annot.png and outimg_legend.png — annotation-legend and biomarker-legend images

## How to apply

Run export2graphlan.py with the abundance table as input, specifying the output tree (-t tree.txt) and annotation (-a annot.txt) files; pass --internal_levels when leaf-to-root abundance sums are needed on internal nodes, as done for the vaginal microbiota temporal-dynamics tree. Choose annotation parameters such as --abundance_threshold (default 20.0), --def_clade_size, and --background_clades/--background_colors to control which clades are highlighted as biomarkers. If the input is in BIOM format, the biom-format library (ver. 2.0.1) must be installed alongside pandas (0.13.1) and SciPy, since BIOM v1/v2 files are accepted directly, enabling QIIME- and mothur-derived tables to be used without manual reformatting. Then feed the two outputs to graphlan_annotate.py to produce the annotated tree (outtree.txt) and render it with GraPhlAn (matplotlib-based) to PNG images, including legend files for annotations and biomarkers. Validate the conversion by checking the six expected files are produced and that the rendered image shows the expected circular taxonomy tree.

## Related tools

- **export2graphlan** (Converts the abundance/biomarker table (text or BIOM) into the GraPhlAn tree.txt and annot.txt files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Annotates the tree via graphlan_annotate.py and renders the circular visualization using matplotlib)
- **hclust2** (Companion tool for heatmap plotting of MetaPhlAn2 abundance profiles in the same pipeline context) — https://github.com/SegataLab/hclust2
- **matplotlib** (Underlying rendering library used by GraPhlAn to draw the output images)

## Examples

```
export2graphlan.py -i abundance.biom --internal_levels -t tree.txt -a annot.txt && graphlan_annotate.py tree.txt annot.txt --annot outtree.txt
```

## Evaluation signals

- Run produces the six expected files: annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt, tree.txt
- outimg.png displays a circular taxonomy tree consistent with the reference figure (e.g. Fig. S3 vaginal microbiota temporal dynamics)
- With --internal_levels, internal node abundances equal leaf-to-root sums of the input table
- Biomarker-highlighting legends (outimg_legend.png) reflect clades passing --abundance_threshold (default 20.0)
- BIOM v1/v2 input parses without manual reformatting when the biom-format library is installed

## Limitations

- Requires specific library versions: pandas 0.13.1, biom-format 2.0.1 (only for BIOM inputs), and SciPy (required by hclust2).
- export2graphlan is versioned 0.2.1 (27 October 2018); the paper does not state software version numbers for GraPhlAn, export2graphlan, or its dependencies.
- Default parameters such as --abundance_threshold 20.0 may filter out low-abundance clades from annotation; adjust for sparse datasets.

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] BIOM prerequisite: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [methods] internal levels flag: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] annotation pipeline: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] BIOM acceptance finding: [evidence span withheld — anchor status: not_found]
- [discussion] rendering library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
