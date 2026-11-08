---
name: metagenomic-biomarker-visualization
description: Use when you have LEfSe biomarker results and a MetaPhlAn (or HUMAnN)
  abundance profile and need a publication-quality circular cladogram highlighting
  discriminatory taxa with metadata overlays.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair — use GraPhlAn directly instead
  of re-converting.
- Interactive exploration of large phylogenies is the goal (iTOL/Krona are the comparison
  points in the paper), not reproducible pipeline rendering.
- The data are not taxonomic (MetaPhlAn), functional (HUMAnN), or biomarker (LEfSe)
  outputs — export2graphlan expects these formats.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN output to GraPhlAn tree.txt and annot.txt,
    mapping LEfSe LDA scores to clade sizes and biomarkers to colors
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into high-quality circular phylogeny images (PNG)
    using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to produce the circular phylogeny
    figures
- name: hclust2
  role: Companion tool for plotting heat-maps of MetaPhlAn2 abundance profiles of
    the same HMP/HMP1-phase2 samples
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:6c5e23207c575248d11cee14d674b0a5d22a18e1a5d19ca8c1ffb762865ed693
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metagenomic-biomarker-visualization

## Summary

Automatically converts MetaPhlAn, LEfSe, and/or HUMAnN outputs into GraPhlAn tree and annotation files, then renders high-quality circular phylogenies (e.g., the HMP aerobiosis cladogram, Fig. S1) with LEfSe biomarker LDA scores mapped to clade sizes and class assignments to shaded backgrounds.

## When to use

Use when you have LEfSe biomarker results and a MetaPhlAn (or HUMAnN) abundance profile and need a publication-quality circular cladogram highlighting discriminatory taxa with metadata overlays.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair — use GraPhlAn directly instead of re-converting.
- Interactive exploration of large phylogenies is the goal (iTOL/Krona are the comparison points in the paper), not reproducible pipeline rendering.
- The data are not taxonomic (MetaPhlAn), functional (HUMAnN), or biomarker (LEfSe) outputs — export2graphlan expects these formats.

## Inputs

- LEfSe input abundance matrix (MetaPhlAn output, e.g. hmp_metahit.txt)
- LEfSe biomarker result file with LDA scores
- Optional BIOM-format abundance files (BIOM >= 2.0.1 supported)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (plain circular phylogeny)
- outimg_annot.png (annotated cladogram)
- outimg_legend.png (biomarker legend)

## How to apply

Run export2graphlan.py on the LEfSe input matrix and LEfSe biomarker result to generate a GraPhlAn tree file (tree.txt) and annotation file (annot.txt), mapping LEfSe biomarker LDA scores to clade sizes (def 10, min 20, max 200) and class assignments to shaded backgrounds via --background_levels or --background_clades. Apply the annotations with graphlan_annotate.py to produce the annotated tree outtree.txt, then render with GraPhlAn (graphlan.py) using matplotlib to obtain outimg.png, outimg_annot.png, and outimg_legend.png. Use --internal_levels to sum abundances from leaf to root on internal nodes, and tune --def_font_size/--min_font_size/--max_font_size (default 10/8/12) for readable labels. The hmp_aerobiosis example wraps these steps in a PIPELINE.sh script, demonstrating the reproducible end-to-end flow.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN output to GraPhlAn tree.txt and annot.txt, mapping LEfSe LDA scores to clade sizes and biomarkers to colors) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into high-quality circular phylogeny images (PNG) using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to produce the circular phylogeny figures)
- **hclust2** (Companion tool for plotting heat-maps of MetaPhlAn2 abundance profiles of the same HMP/HMP1-phase2 samples) — https://github.com/SegataLab/hclust2

## Examples

```
cd hmp_aerobiosis && ./PIPELINE.sh && graphlan_annotate.py annot.txt tree.txt --annotations outtree.txt && graphlan.py outtree.txt outimg_annot.png
```

## Evaluation signals

- All six expected files (annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png) are present in the example folder after running PIPELINE.sh
- outimg_annot.png shows the annotated cladogram with shaded background regions corresponding to LEfSe biomarker class assignments
- Clade node sizes in the rendering are monotonic with LEfSe LDA scores, bounded by --min_clade_size (20) and --max_clade_size (200)
- tree.txt is valid Newick-format tree input and outtree.txt is a superset of tree.txt with annotation blocks appended
- Regeneration of the hmp_aerobiosis cladogram (Fig. S1) is visually consistent with the published figure

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM) beyond export2graphlan ver. 0.2.1 (27 October 2018) and pandas 0.13.1 / BIOM 2.0.1 prerequisites
- No changelog is found in the repository documentation
- BIOM library is required only if input files are in BIOM format; SciPy is required by hclust2
- Very large phylogenies with thousands of taxa may be better targeted at interactive tools (iTOL, Krona) if interactivity, rather than compact reproducible rendering, is required

## Evidence

- [methods] export2graphlan.py converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and an annotation file (annot.txt): [evidence span withheld — anchor status: not_found]
- [discussion] GraPhlAn uses the matplotlib library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] export2graphlan converts MetaPhlAn/LEfSe/HUMAnN outputs: "export2graphlan.py (ver. 0.2.1 of 27 October 2018). Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format."
- [readme] default clade size parameters: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [methods] outtree.txt is produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
