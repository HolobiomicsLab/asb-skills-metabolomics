---
name: circular-phylogeny-visualization
description: Use when you need a publication-quality circular visualization of a taxonomic/phylogenetic
  tree (e.g., BIOM abundance tables from 16S studies, MetaPhlAn/LEfSe/HUMAnN outputs)
  annotated with metadata or biomarkers.
when_to_use_negative:
- Input is already a GraPhlAn-annotated tree or the goal is interactive phylogeny
  exploration better served by iTOL or Krona.
- The goal is a heatmap of sample-vs-feature abundance profiles rather than a circular
  taxonomy tree (use hclust2 instead).
- Input profiles are not hierarchical taxonomic/metabolic tables (e.g., raw reads
  or non-tree-structured matrices without taxonomy lineage strings).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: GraPhlAn
  role: Renders the annotated tree file into a high-quality circular phylogeny image
    using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, HUMAnN, or BIOM abundance inputs into the GraPhlAn
    tree.txt and annot.txt files
  repo: https://github.com/SegataLab/export2graphlan
- name: matplotlib
  role: Graphics library GraPhlAn uses for plotting
- name: Galaxy
  role: Web application through which GraPhlAn can be executed online
- name: hclust2
  role: Companion tool for heatmap visualization of abundance profiles (e.g., MetaPhlAn2
    HMP profiles); SciPy dependency for export2graphlan environment
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:fb811f74cf0df8313087ec36b7f400cd57444400f6e766802090941fe15da5af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# circular-phylogeny-visualization

## Summary

Generate high-quality, compact circular phylogenetic/taxonomy trees with GraPhlAn by converting abundance or biomarker tables into a tree file and annotation file via export2graphlan, annotating with graphlan_annotate.py, and rendering with GraPhlAn (matplotlib-based).

## When to use

Use when you need a publication-quality circular visualization of a taxonomic/phylogenetic tree (e.g., BIOM abundance tables from 16S studies, MetaPhlAn/LEfSe/HUMAnN outputs) annotated with metadata or biomarkers.

## When NOT to use

- Input is already a GraPhlAn-annotated tree or the goal is interactive phylogeny exploration better served by iTOL or Krona.
- The goal is a heatmap of sample-vs-feature abundance profiles rather than a circular taxonomy tree (use hclust2 instead).
- Input profiles are not hierarchical taxonomic/metabolic tables (e.g., raw reads or non-tree-structured matrices without taxonomy lineage strings).

## Inputs

- BIOM-format abundance matrix (versions 1 and 2), e.g., from QIIME or mothur 16S rRNA profiling
- MetaPhlAn/MetaPhlAn2 taxonomic profile table (e.g., hmp_metahit.txt)
- LEfSe biomarkers output (LEfSe input/output files)
- HUMAnN functional profile (KEGG ortholog groups)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (rendered circular taxonomy tree)
- outimg_annot.png (annotation legend image)
- outimg_legend.png (highlighted-biomarker legend image)

## How to apply

Run export2graphlan.py on the input abundance matrix (e.g., a BIOM-format table from a 16S vaginal microbiota study, supporting BIOM versions 1 and 2) with --internal_levels so abundances are summed from leaf to root on internal nodes, producing tree.txt and annot.txt. Annotate the generated tree with graphlan_annotate.py to produce the annotated outtree.txt, then render it with GraPhlAn (which uses matplotlib) to draw the circular tree as outimg.png plus legend images (outimg_annot.png, outimg_legend.png). Tune visualization parameters such as --title, --def_clade_size/--min_clade_size/--max_clade_size, font sizes, and --background_clades with --background_colors to highlight biomarker clades. export2graphlan requires pandas (0.13.1), BIOM (2.0.1, only for BIOM inputs), and SciPy (required by hclust2), and is installable via Bioconda or PyPI.

## Related tools

- **GraPhlAn** (Renders the annotated tree file into a high-quality circular phylogeny image using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **export2graphlan** (Converts MetaPhlAn, LEfSe, HUMAnN, or BIOM abundance inputs into the GraPhlAn tree.txt and annot.txt files) — https://github.com/SegataLab/export2graphlan
- **matplotlib** (Graphics library GraPhlAn uses for plotting)
- **Galaxy** (Web application through which GraPhlAn can be executed online)
- **hclust2** (Companion tool for heatmap visualization of abundance profiles (e.g., MetaPhlAn2 HMP profiles); SciPy dependency for export2graphlan environment) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i abundances.biom --internal_levels -t tree.txt -a annot.txt && graphlan_annotate.py tree.txt outtree.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- Run produces exactly six expected files: annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt, tree.txt.
- outimg.png displays a circular taxonomy tree consistent with the reference figure (e.g., the Fig. S3 vaginal microbiota temporal-dynamics tree).
- With --internal_levels, internal-node abundances equal the leaf-to-root sums of the input abundance matrix.
- Tree and annotation files parse under GraPhlAn without errors, and the annotation legend images match the biomarker clades supplied.
- Highlighted biomarker clades in outimg_legend.png correspond to the LEfSe biomarkers in the input annotation.

## Limitations

- No software version numbers are stated in the article for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM); the README lists export2graphlan ver. 0.2.1 (27 October 2018) with pandas 0.13.1 and BIOM 2.0.1 as prerequisites.
- No changelog is found in the repository documentation.
- Rendering quality and clade sizing depend on thresholds/parameters (e.g., --abundance_threshold default 20.0, --most_abundant, --least_biomarkers) that may need tuning per dataset.

## Evidence

- [other] BIOM support finding: [evidence span withheld — anchor status: not_found]
- [other] workflow step: [evidence span withheld — anchor status: not_found]
- [methods] annotation step: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] rendering library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] prerequisites: [evidence span withheld — anchor status: not_found]
- [abstract] tool purpose: "We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes."
- [readme] internal_levels flag: "--internal_levels     If specified sum-up from leaf to root the abundances values."
