---
name: lefse-output-to-graphlan-conversion
description: Use when you have MetaPhlAn/LEfSe (or HUMAnN) input and result files
  and need to generate publication-quality circular cladograms highlighting LEfSe
  biomarker clades with LDA-mapped clade sizes and shaded background classes.
when_to_use_negative:
- The input is already a GraPhlAn tree/annotation pair — run graphlan_annotate.py
  directly instead of re-converting.
- The goal is an interactive exploration of a large phylogeny rather than a static
  publication figure (tools like iTOL or Krona target interactive analyses).
- Input data are not from MetaPhlAn, LEfSe, or HUMAnN and do not encode taxonomic/functional
  hierarchies with biomarker statistics.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0654
tools:
- name: export2graphlan
  role: Converts LEfSe input matrix and biomarker result into GraPhlAn tree.txt and
    annot.txt
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into circular phylogeny images using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn for figure rendering
- name: hclust2
  role: Dependency for export2graphlan (provides SciPy-based matrix handling) and
    example heatmap plotting of MetaPhlAn2 profiles
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:de8218475b3c3aaed1fcb6d02b121a726df373312bf01c8c2ea943f9bd2530e8
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
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# lefse-output-to-graphlan-conversion

## Summary

Convert LEfSe biomarker inputs and results into GraPhlAn tree (tree.txt) and annotation (annot.txt) files using export2graphlan.py, then render them as compact circular cladograms (e.g., the HMP aerobiosis Fig. S1 or HMP-vs-MetaHIT Fig. 3 plots) via graphlan_annotate.py and GraPhlAn's matplotlib backend.

## When to use

Use when you have MetaPhlAn/LEfSe (or HUMAnN) input and result files and need to generate publication-quality circular cladograms highlighting LEfSe biomarker clades with LDA-mapped clade sizes and shaded background classes.

## When NOT to use

- The input is already a GraPhlAn tree/annotation pair — run graphlan_annotate.py directly instead of re-converting.
- The goal is an interactive exploration of a large phylogeny rather than a static publication figure (tools like iTOL or Krona target interactive analyses).
- Input data are not from MetaPhlAn, LEfSe, or HUMAnN and do not encode taxonomic/functional hierarchies with biomarker statistics.

## Inputs

- LEfSe input matrix (taxonomic abundance table from MetaPhlAn2 profiling, e.g., hmp_metahit.txt)
- LEfSe biomarker result file
- Optionally, HUMAnN functional profiles (KEGG ortholog groups) or BIOM-format abundance files

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file mapping LDA scores to clade sizes and classes to shaded backgrounds)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png, outimg_annot.png, outimg_legend.png (circular phylogeny PNG renders via matplotlib)

## How to apply

Run export2graphlan.py with -i/-o pointing at the LEfSe input matrix and LEfSe result, and -t tree.txt / -a annot.txt as the output tree and annotation files; optionally set --annotations, --background_levels (or --background_clades), and clade/font size ranges (--def_clade_size 10, --min_clade_size 20, --max_clade_size 200) so LEfSe LDA scores map to clade sizes and class assignments map to shaded backgrounds. Apply annot.txt to the tree with graphlan_annotate.py to produce the annotated outtree.txt. Render outtree.txt with GraPhlAn to obtain outimg.png (plain tree), outimg_annot.png (tree with annotation legend), and outimg_legend.png (biomarker legend). export2graphlan (ver. 0.2.1) requires pandas 0.13.1, SciPy, and BIOM 2.0.1 only for BIOM-format inputs, and optionally sums abundances from leaf to root on internal nodes via --internal_levels.

## Related tools

- **export2graphlan** (Converts LEfSe input matrix and biomarker result into GraPhlAn tree.txt and annot.txt) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into circular phylogeny images using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn for figure rendering)
- **hclust2** (Dependency for export2graphlan (provides SciPy-based matrix handling) and example heatmap plotting of MetaPhlAn2 profiles) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_result.txt -t tree.txt -a annot.txt; graphlan_annotate.py tree.txt annot.txt outtree.txt; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six expected output files exist in the example folder: annot.txt, tree.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png.
- outimg_annot.png renders a valid PNG circular cladogram with the annotation legend visible.
- outimg.png is a valid PNG circular tree with highlighted HMP- vs. MetaHIT-enriched clades (for the hmp_metahit example).
- LEfSe biomarker LDA scores map to clade node sizes and class assignments map to shaded background regions in the rendered tree.
- Run in the hmp_aerobiosis example folder via ./PIPELINE.sh, the pipeline completes without errors and regenerates the Fig. S1-style aerobiosis cladogram.

## Limitations

- No software version numbers are stated in the article for GraPhlAn itself or its dependencies beyond export2graphlan ver. 0.2.1 (27 October 2018), pandas 0.13.1, and BIOM 2.0.1; no changelog is available in the repository documentation.
- BIOM support requires biom-format ver. 2.0.1; without it, BIOM-format inputs cannot be processed.
- Defaults such as --abundance_threshold 20.0 and clade size range 20–200 may need tuning per dataset; LEfSe must already have been run to supply the biomarker result file.

## Evidence

- [methods] export2graphlan.py converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and an annotation file (annot.txt): [evidence span withheld — anchor status: not_found]
- [readme] README usage description: "**export2graphlan** is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [methods] annotation directives step: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib backend: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] internal levels option: "--internal_levels     If specified sum-up from leaf to root the abundances values."
