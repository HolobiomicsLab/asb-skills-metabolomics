---
name: microbiome-taxonomic-cladogram-annotation
description: Use when you have MetaPhlAn or LEfSe biomarker results (e.g., 16S rRNA
  profiles from hmp_aerobiosis) and need a compact, publication-quality circular phylogeny
  with clade-level biomarker highlighting and shaded backgrounds.
when_to_use_negative:
- When the tree and annotations already exist in GraPhlAn input format — export2graphlan
  is a converter, not a plotter; use GraPhlAn directly
- When interactive exploration of large-scale phylogenies is the goal — iTOL targets
  interactive analyses and Krona emphasizes hierarchical quantitative information,
  whereas GraPhlAn is command-driven for pipeline reproducibility
- For functional (HUMAnN/KEGG) heatmaps rather than taxonomic cladograms — hclust2
  is the matching tool for heat-maps of abundance profiles
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN output into GraPhlAn tree.txt and
    annot.txt, mapping LEfSe LDA scores to clade sizes and class assignments to shaded
    backgrounds
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into high-quality circular phylogenies (outimg.png,
    outimg_annot.png, outimg_legend.png) using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Rendering library underlying GraPhlAn image generation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:b0ee13c689c2be28840873e47756adba528e828e869cf277ed5ccca46f450493
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# microbiome-taxonomic-cladogram-annotation

## Summary

Generates annotated circular taxonomic cladograms (e.g., the HMP aerobiosis cladogram, Fig. S1) by converting MetaPhlAn/LEfSe outputs into GraPhlAn tree and annotation files with export2graphlan, then rendering them with GraPhlAn/matplotlib. LEfSe biomarker LDA scores map to clade sizes and class assignments to shaded backgrounds.

## When to use

Use when you have MetaPhlAn or LEfSe biomarker results (e.g., 16S rRNA profiles from hmp_aerobiosis) and need a compact, publication-quality circular phylogeny with clade-level biomarker highlighting and shaded backgrounds.

## When NOT to use

- When the tree and annotations already exist in GraPhlAn input format — export2graphlan is a converter, not a plotter; use GraPhlAn directly
- When interactive exploration of large-scale phylogenies is the goal — iTOL targets interactive analyses and Krona emphasizes hierarchical quantitative information, whereas GraPhlAn is command-driven for pipeline reproducibility
- For functional (HUMAnN/KEGG) heatmaps rather than taxonomic cladograms — hclust2 is the matching tool for heat-maps of abundance profiles

## Inputs

- LEfSe input matrix (taxonomic abundance table, e.g., from MetaPhlAn/MetaPhlAn2 profiling)
- LEfSe biomarker result file with LDA scores
- Optionally, BIOM-format input files (requires biom-format ver. 2.0.1)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree produced by graphlan_annotate.py)
- outimg.png (plain circular phylogeny)
- outimg_annot.png (annotated cladogram with legend)
- outimg_legend.png (biomarker legend)

## How to apply

Run the example PIPELINE.sh (e.g., $ ./PIPELINE.sh in the hmp_aerobiosis folder) so export2graphlan.py converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and annotation file (annot.txt), mapping LEfSe biomarker LDA scores to clade sizes (--min_clade_size 20, --max_clade_size 200, default 10 for non-biomarkers) and class assignments to shaded backgrounds (--background_levels). Then run graphlan_annotate.py applying annot.txt to tree.txt to produce the annotated outtree.txt. Finally render outtree.txt with GraPhlAn, which uses matplotlib, into outimg.png (plain tree), outimg_annot.png (tree with annotation legend), and outimg_legend.png (biomarker legend). export2graphlan requires pandas (ver. 0.13.1), BIOM 2.0.1 (only for BIOM input), and SciPy (required by hclust2).

## Related tools

- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN output into GraPhlAn tree.txt and annot.txt, mapping LEfSe LDA scores to clade sizes and class assignments to shaded backgrounds) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into high-quality circular phylogenies (outimg.png, outimg_annot.png, outimg_legend.png) using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Rendering library underlying GraPhlAn image generation)

## Examples

```
$ ./PIPELINE.sh
```

## Evaluation signals

- All six expected output files present in the example folder: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg_annot.png visually shows the annotated cladogram with the annotation legend
- outimg_legend.png contains the LEfSe biomarker legend with biomarker-to-color mappings
- Clade sizes in tree.txt scale with LEfSe LDA scores within the configured min/max clade size bounds (defaults 20 and 200)
- Shaded backgrounds in the rendering correspond to LEfSe class assignments at the specified background levels

## Limitations

- No software version numbers stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM); export2graphlan.py is ver. 0.2.1 of 27 October 2018
- export2graphlan requires pandas 0.13.1, BIOM 2.0.1 (only for BIOM inputs), and SciPy as prerequisites
- No changelog found in the repository documentation
- Paths must be configured (export PATH=`pwd`/export2graphlan/:$PATH) so export2graphlan is callable from anywhere

## Evidence

- [other] export2graphlan.py converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and an annotation file (annot.txt): [evidence span withheld — anchor status: not_found]
- [methods] examples folder sub-folders: "The ``examples`` folder contains the following sub-folders: ``hmp_aerobiosis``, ``hmp_metahit_metabolic``, and ``hmp_metahit_mp2``."
- [methods] annotated tree production: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib dependency: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] prerequisite libraries: [evidence span withheld — anchor status: not_found]
- [readme] clade size defaults: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
