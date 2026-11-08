---
name: graphlan-tree-annotation-and-rendering
description: Use when LEfSe biomarker results or MetaPhlAn/HUMAnN taxonomic-functional
  profiles must be rendered as a compact circular cladogram with highlighted sub-trees,
  sizes mapped to biomarker effect, and legends.
when_to_use_negative:
- The input is an interactive-exploration use case better served by iTOL or Krona,
  which the paper positions as targeting interactive analyses or hierarchical quantitative
  browsing rather than command-driven publication rendering
- The data are not phylogenetic/taxonomic trees with metadata (GraPhlAn targets microbial
  phylogenies, taxonomic profiles, and biomarker cladograms, though it has been applied
  to animal and plant taxonomies)
- Only a raw feature-abundance table is available with no LEfSe biomarker result and
  no intent to identify differentially abundant clades
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe output into the GraPhlAn tree.txt and annot.txt files,
    automatically highlighting biomarker sub-trees
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Annotates the tree (graphlan_annotate.py) and renders the final circular cladogram
    PNG via its matplotlib backend
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to draw the circular phylogeny
- name: Galaxy
  role: Optional web interface for executing GraPhlAn locally-equivalent analyses
    online
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:2c33b9e1299516b4c2b955856afb825bd88c238f2b8e5ac7cb4a7f00dce15949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - Galaxy
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graphlan-tree-annotation-and-rendering

## Summary

Reproduce publication-quality circular phylogenetic cladograms (e.g., the HMP vs. MetaHIT LEfSe biomarker figure) by converting MetaPhlAn/LEfSe output to GraPhlAn input with export2graphlan, annotating the tree with graphlan_annotate.py, and rendering with GraPhlAn's matplotlib backend.

## When to use

Use when LEfSe biomarker results or MetaPhlAn/HUMAnN taxonomic-functional profiles must be rendered as a compact circular cladogram with highlighted sub-trees, sizes mapped to biomarker effect, and legends.

## When NOT to use

- The input is an interactive-exploration use case better served by iTOL or Krona, which the paper positions as targeting interactive analyses or hierarchical quantitative browsing rather than command-driven publication rendering
- The data are not phylogenetic/taxonomic trees with metadata (GraPhlAn targets microbial phylogenies, taxonomic profiles, and biomarker cladograms, though it has been applied to animal and plant taxonomies)
- Only a raw feature-abundance table is available with no LEfSe biomarker result and no intent to identify differentially abundant clades

## Inputs

- LEfSe input file (MetaPhlAn taxonomic abundance table formatted for LEfSe, e.g., hmp_metahit.txt)
- LEfSe output file (biomarker analysis result)
- Optional BIOM-format abundance tables (with biom-format ver. 2.0.1 installed)

## Outputs

- tree.txt (GraPhlAn tree file from export2graphlan)
- annot.txt (GraPhlAn annotation file from export2graphlan)
- outtree.txt (annotated tree produced by graphlan_annotate.py)
- outimg.png (rendered circular cladogram PNG)
- outimg_annot.png (annotation-legend render)
- outimg_legend.png (biomarker-legend render)

## How to apply

Start from the LEfSe input and output files for the filtered dataset (e.g., the healthy-individual HMP vs. MetaHIT comparison in the hmp_metahit example folder) and run export2graphlan.py with -i/-o pointing at those files and -t tree.txt / -a annot.txt as outputs; export2graphlan automatically determines which nodes are biomarkers and scales clade radii between --min_clade_size (20) and --max_clade_size (200) versus --def_clade_size (10) for non-biomarkers, with font sizes bounded by --min_font_size (8) and --max_font_size (12). Optionally tune --background_clades, --background_colors (RGB or HSV), --title, and --abundance_threshold (default 20.0) to control which sub-trees and shaded backgrounds are emphasized. Annotate the generated tree with graphlan_annotate.py to merge the annotation directives into outtree.txt, then execute GraPhlAn (graphlan.py), which uses the matplotlib library, on outtree.txt to render the circular cladogram as a PNG (outimg.png) plus companion annotation-legend and biomarker-legend renders (outimg_annot.png, outimg_legend.png). Rationale: the two-step convert-then-annotate design keeps tree generation, annotation, and rendering separable so each stage is inspectable and re-runnable inside a publication pipeline.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe output into the GraPhlAn tree.txt and annot.txt files, automatically highlighting biomarker sub-trees) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Annotates the tree (graphlan_annotate.py) and renders the final circular cladogram PNG via its matplotlib backend) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to draw the circular phylogeny)
- **Galaxy** (Optional web interface for executing GraPhlAn locally-equivalent analyses online)

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt && graphlan_annotate.py annot.txt tree.txt outtree.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six expected example outputs exist: annot.txt, tree.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a valid PNG depicting a circular tree with highlighted HMP- vs. MetaHIT-enriched clades
- The annotation file distinguishes biomarker clades (sized between --min_clade_size 20 and --max_clade_size 200) from non-biomarkers rendered at --def_clade_size 10
- outtree.txt contains the annotation directives attached by graphlan_annotate.py on top of the raw tree.txt
- Legend renders (outimg_annot.png, outimg_legend.png) are produced alongside the main plot, matching the pipeline's expected outputs

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or its dependencies (matplotlib, pandas, BIOM) in the documentation; export2graphlan.py is ver. 0.2.1 of 27 October 2018
- export2graphlan requires specific library versions (pandas ver. 0.13.1, BIOM ver. 2.0.1 for BIOM inputs, SciPy via hclust2)
- Biomarker clade sizing and annotation depend on LEfSe output quality and the --abundance_threshold (default 20.0), so low-abundance biomarkers may be dropped or unsized

## Evidence

- [other] The HMP-vs-MetaHIT biomarker cladogram in Fig. 3 was generated with the export2graphlan script provided with GraPhlAn, after LEfSe analysis of the filtered dataset of healthy-individual samples.: [evidence span withheld — anchor status: not_found]
- [readme] Export2graphlan produces tree and annotation files consumed by graphlan_annotate.py and graphlan.py.: "The two output file of **export2graphlan** should then be used to run ``graphlan_annotate.py``, in order to attach to the tree the derived annotations, and finally, by executing ``graphlan.py`` the"
- [discussion] GraPhlAn uses matplotlib as its rendering backend.: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] Workflow methods name the annotated tree artifact and the conversion step.: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [readme] Clade size parameters have documented defaults bounding biomarker vs non-biomarker rendering.: "Set the minimum value of clades that are biomarkers. Default is 20"
