---
name: lefse-biomarker-annotation
description: Use when you have a LEfSe-format abundance matrix (from merged MetaPhlAn
  or HUMAnN profiles) plus a LEfSe result file and need to render a GraPhlAn circular
  tree that highlights biomarker clades.
when_to_use_negative:
- Your input is already a GraPhlAn tree/annotation pair or a Newick tree with no LEfSe
  biomarker analysis — export2graphlan is specifically a converter for MetaPhlAn/LEfSe/HUMAnN
  outputs.
- You need an interactive tree explorer rather than a static publication-quality circular
  figure (the article positions GraPhlAn against iTOL/Krona for pipeline-embedded,
  command-driven reproducibility).
- The abundance data has no taxonomic/functional hierarchy (e.g. a flat feature table),
  since clade-level sizing, internal-level summation, and sub-tree highlighting depend
  on hierarchical clade names.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN output into the GraPhlAn tree.txt and annot.txt
    files, scaling biomarker clades by LDA-derived importance
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated circular phylogenetic/functional tree via graphlan_annotate.py
    and graphlan.py
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn to render the final image
- name: hclust2
  role: Companion SciPy-dependent tool for heatmap visualization of MetaPhlAn2 abundance
    profiles in the same pipeline ecosystem
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:1eb117833981d0cf63115626cd90ab5d1e085d2d1e0f65db95bcacde730806cf
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
yes_rate: 0.7142857142857143
n_claims: 14
---

# lefse-biomarker-annotation

## Summary

Converts MetaPhlAn/LEfSe (and HUMAnN) outputs into GraPhlAn tree and annotation files using export2graphlan, so that clades identified as LEfSe biomarkers are automatically sized, colored, and highlighted in the rendered phylogenetic or functional tree.

## When to use

Use when you have a LEfSe-format abundance matrix (from merged MetaPhlAn or HUMAnN profiles) plus a LEfSe result file and need to render a GraPhlAn circular tree that highlights biomarker clades.

## When NOT to use

- Your input is already a GraPhlAn tree/annotation pair or a Newick tree with no LEfSe biomarker analysis — export2graphlan is specifically a converter for MetaPhlAn/LEfSe/HUMAnN outputs.
- You need an interactive tree explorer rather than a static publication-quality circular figure (the article positions GraPhlAn against iTOL/Krona for pipeline-embedded, command-driven reproducibility).
- The abundance data has no taxonomic/functional hierarchy (e.g. a flat feature table), since clade-level sizing, internal-level summation, and sub-tree highlighting depend on hierarchical clade names.

## Inputs

- LEfSe-format abundance matrix (merged MetaPhlAn v1/v2 taxonomic profiles or HUMAnN functional profiles) as -i/--lefse_input
- LEfSe result file listing biomarker clades with LDA scores as -o/--lefse_output
- Optional BIOM-format input files (requires biom-format 2.0.1)

## Outputs

- tree.txt — GraPhlAn tree file containing all input clades
- annot.txt — GraPhlAn annotation file with ring colors/highlighting for LEfSe biomarker clades
- outtree.txt — annotated tree produced by graphlan_annotate.py
- Rendered circular tree images (e.g. outimg.png, outimg_annot.png, outimg_legend.png)

## How to apply

Run export2graphlan.py with the LEfSe input abundance matrix as -i and the LEfSe result as -o, emitting a tree file (-t tree.txt) and an annotation file (-a annot.txt). Biomarker clades scale in size between --min_clade_size 20 and --max_clade_size 200 while non-biomarkers get --def_clade_size 10, and font sizes default to min 8, max 12, default 10. Enable --internal_levels to sum abundances leaf-to-root on internal taxonomy nodes before highlighting; restrict annotation to clades above the abundance threshold (default 20.0) and to the most abundant clades (default 10) when biomarker labels are absent. Then annotate the tree with graphlan_annotate.py to produce outtree.txt and render the final circular image with GraPhlAn (matplotlib-backed), reproducing e.g. the HMP vs MetaHIT KEGG functional tree of Fig. 4 where Glycolysis and Citrate cycle enrich in HMP and Sulfur/Vitamin B6 Metabolism enrich in MetaHIT.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN output into the GraPhlAn tree.txt and annot.txt files, scaling biomarker clades by LDA-derived importance) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated circular phylogenetic/functional tree via graphlan_annotate.py and graphlan.py) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn to render the final image)
- **hclust2** (Companion SciPy-dependent tool for heatmap visualization of MetaPhlAn2 abundance profiles in the same pipeline ecosystem) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --internal_levels && graphlan_annotate.py annot.txt tree.txt --annotations outtree.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- Both tree.txt and annot.txt exist and are non-empty after running export2graphlan.py.
- tree.txt parses as a GraPhlAn tree with all input clades represented.
- annot.txt contains annotation entries (ring colors/highlighting) for every LEfSe biomarker clade listed in the LEfSe result file.
- For the PIPELINE.sh workflow, all six expected files exist (annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png) and outimg.png is a non-empty rendered circular tree.
- Biomarker clades receive clade sizes within the 20–200 range and non-biomarkers the default size 10, matching the --min_clade_size/--max_clade_size/--def_clade_size settings.

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM) in the article documentation; the README states export2graphlan ver. 0.2.1 and pandas 0.13.1.
- BIOM support requires biom-format ver. 2.0.1 and hclust2 requires SciPy, so missing dependencies break BIOM-format inputs.
- When biomarker labels are absent, annotation falls back to abundance-based restriction (threshold 20.0, top 10 most abundant clades), which may not reflect differential biomarkers.
- No changelog is found in the repository documentation.

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] Biomarker clade sizing defaults: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [methods] internal_levels option: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] PIPELINE.sh example workflow: "The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree."
- [intro] Command-driven design rationale: "Second, visualization tools must be command-driven in order to be embedded in computational pipelines."
