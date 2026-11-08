---
name: circular-taxonomic-tree-rendering
description: Use when you need a publication-quality circular cladogram of a microbial
  taxonomy (e.g., HMP aerobiosis LEfSe biomarkers or saliva BIOM tables), mapping
  LDA scores to clade sizes and class assignments to shaded backgrounds.
when_to_use_negative:
- Input is a generic Newick phylogeny without taxonomic abundance/LEfSe annotations
  to map — plain tree viewers may suffice
- The goal is interactive exploration of hierarchical quantitative profiles rather
  than a static publication figure (Krona targets interactive analyses)
- Input is a raw FASTQ or unprofiled reads — taxonomic/functional profiling with MetaPhlAn2
  or HUMAnN must happen first
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts LEfSe/MetaPhlAn/HUMAnN/BIOM inputs into GraPhlAn tree and annotation
    files, mapping LDA scores to clade sizes and class assignments to shaded backgrounds
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into high-quality circular phylogeny images with
    annotation and biomarker legends
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn to produce the final circular
    figures
- name: hclust2
  role: Companion heatmap tool for the clustered MetaPhlAn2 abundance profiles that
    accompany the GraPhlAn workflow
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:049d7e8021cae15ad8df700cb721ba2a5ea21c0abd8599808250ffc6ae155009
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# circular-taxonomic-tree-rendering

## Summary

Renders compact circular taxonomic/phylogenetic trees with GraPhlAn by converting MetaPhlAn, LEfSe, HUMAnN, or BIOM abundance tables into GraPhlAn tree and annotation files via export2graphlan, then annotating and plotting the tree with matplotlib-backed GraPhlAn.

## When to use

Use when you need a publication-quality circular cladogram of a microbial taxonomy (e.g., HMP aerobiosis LEfSe biomarkers or saliva BIOM tables), mapping LDA scores to clade sizes and class assignments to shaded backgrounds.

## When NOT to use

- Input is a generic Newick phylogeny without taxonomic abundance/LEfSe annotations to map — plain tree viewers may suffice
- The goal is interactive exploration of hierarchical quantitative profiles rather than a static publication figure (Krona targets interactive analyses)
- Input is a raw FASTQ or unprofiled reads — taxonomic/functional profiling with MetaPhlAn2 or HUMAnN must happen first

## Inputs

- LEfSe input abundance matrix (e.g., hmp_aerobiosis LEfSe-style input)
- LEfSe biomarker result file with LDA scores and class assignments
- BIOM-format 16S abundance table (e.g., QIIME/mothur saliva microbiome profile)
- MetaPhlAn or HUMAnN output profiles (optionally merged, e.g., hmp_metahit.txt)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (plain circular tree image)
- outimg_annot.png (annotated cladogram with annotation legend)
- outimg_legend.png (biomarker legend image)

## How to apply

Add the export2graphlan directory to PATH (export PATH=`pwd`/export2graphlan/:$PATH) so the scripts are callable. Run export2graphlan.py with a LEfSe-style input matrix or BIOM file (-i), requesting a tree file (-t tree.txt) and annotation file (-a annot.txt); defaults map LEfSe biomarker LDA scores to clade sizes (def_clade_size 10, min_clade_size 20, max_clade_size 200) and font sizes (def 10, min 8, max 12). Annotate the produced tree with graphlan_annotate.py to obtain the annotated tree (outtree.txt), attaching biomarker highlights and shaded class backgrounds. Render with GraPhlAn (matplotlib-backed) to obtain the plain tree image, the annotated image, and the biomarker legend image. Rationale: export2graphlan automatically converts outputs of metagenomic profiling tools (MetaPhlAn, HUMAnN, LEfSe), and GraPhlAn produces compact visualizations spanning up to thousands of taxa with overlaid metadata.

## Related tools

- **export2graphlan** (Converts LEfSe/MetaPhlAn/HUMAnN/BIOM inputs into GraPhlAn tree and annotation files, mapping LDA scores to clade sizes and class assignments to shaded backgrounds) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into high-quality circular phylogeny images with annotation and biomarker legends) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn to produce the final circular figures)
- **hclust2** (Companion heatmap tool for the clustered MetaPhlAn2 abundance profiles that accompany the GraPhlAn workflow) — https://github.com/SegataLab/hclust2

## Examples

```
export PATH=`pwd`/export2graphlan/:$PATH; export2graphlan.py -i saliva_microbiome.biom -t tree.txt -a annot.txt --internal_levels; graphlan_annotate.py tree.txt annot.txt --annot outtree.txt; graphlan.py outtree.txt outimg_annot.png
```

## Evaluation signals

- All six expected outputs exist: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a non-empty circular tree figure matching the reference figure (e.g., Fig. S1 aerobiosis cladogram or Fig. S2 saliva plot)
- tree.txt and annot.txt are produced by export2graphlan.py before annotation, and outtree.txt differs from tree.txt by attached annotations
- Clade sizes fall between min_clade_size (20) and max_clade_size (200) for LEfSe biomarkers, with non-biomarker clades at def_clade_size (10)
- The BIOM path renders a tree consistent with the table: BIOM input is accepted directly by export2graphlan without format conversion

## Limitations

- Requires pandas 0.13.1, and BIOM 2.0.1 only if input files are in BIOM format; SciPy is required by hclust2
- No software version numbers are stated for GraPhlAn or export2graphlan, and no changelog is available, complicating reproducibility pinning
- Biomarker-to-size mapping is capped by max_clade_size (default 200) and clades below the default abundance threshold (20.0) are not annotated

## Evidence

- [results] export2graphlan automatically converts the outputs of metagenomic analysis tools (MetaPhlAn, HUMAnN, LEfSe) into GraPhlAn input files: [evidence span withheld — anchor status: not_found]
- [abstract] GraPhlAn produces high-quality, compact visualizations: "We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes."
- [methods] outtree.txt is produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [readme] export2graphlan requires pandas, BIOM for BIOM input files, and SciPy: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [readme] export2graphlan.py converts MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format: "export2graphlan.py (ver. 0.2.1 of 27 October 2018). Convert MetaPhlAn, LEfSe,
and/or HUMAnN output to GraPhlAn input format."
