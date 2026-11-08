---
name: microbiome-temporal-dynamics-visualization
description: Use when the input is a BIOM (version 1 or 2) abundance matrix from a
  16S rRNA sequencing study (e.g., vaginal microbiota temporal-dynamics data) and
  a compact circular phylogenetic/taxonomy visualization with highlighted biomarkers
  is needed.
when_to_use_negative:
- Input is already a GraPhlAn-formatted tree/annotation file pair rather than a BIOM
  or MetaPhlAn/LEfSe/HUMAnN table — export2graphlan conversion is unnecessary
- The visualization target is an interactive exploration of a large phylogeny rather
  than a compact publication-quality figure — tools like iTOL or Krona target interactive
  analyses
- The abundance table is not in BIOM format and no taxonomy hierarchy (leaf-to-root
  lineages) is present, since --internal_levels requires summing abundances over taxonomic
  levels
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0610
tools:
- name: export2graphlan
  role: Converts BIOM abundance input into the GraPhlAn tree.txt and annot.txt files,
    summing leaf-to-root abundances with --internal_levels
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Annotates (graphlan_annotate.py) and renders the circular taxonomy tree and
    legend images via matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Rendering library used by GraPhlAn to draw the circular tree and legend figures
- name: BIOM
  role: biom-format library (ver. 2.0.1) required by export2graphlan only when input
    files are in BIOM format
  repo: http://biom-format.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:874a2016ebade6c8718cfbddb4fd8dfaaa9f13fe73a87474be17c4c28ab190c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - BIOM
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# microbiome-temporal-dynamics-visualization

## Summary

Converts BIOM-format 16S rRNA abundance tables (e.g., vaginal microbiota profiles) into a GraPhlAn circular taxonomy tree using export2graphlan, graphlan_annotate.py, and GraPhlAn (matplotlib-based rendering).

## When to use

Use when the input is a BIOM (version 1 or 2) abundance matrix from a 16S rRNA sequencing study (e.g., vaginal microbiota temporal-dynamics data) and a compact circular phylogenetic/taxonomy visualization with highlighted biomarkers is needed.

## When NOT to use

- Input is already a GraPhlAn-formatted tree/annotation file pair rather than a BIOM or MetaPhlAn/LEfSe/HUMAnN table — export2graphlan conversion is unnecessary
- The visualization target is an interactive exploration of a large phylogeny rather than a compact publication-quality figure — tools like iTOL or Krona target interactive analyses
- The abundance table is not in BIOM format and no taxonomy hierarchy (leaf-to-root lineages) is present, since --internal_levels requires summing abundances over taxonomic levels

## Inputs

- BIOM-format abundance matrix (BIOM version 1 or 2, e.g., produced by QIIME or mothur from 16S rRNA sequencing)
- export2graphlan.py command-line options (e.g., --internal_levels)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (circular taxonomy tree rendered by GraPhlAn)
- outimg_annot.png (annotation-legend image)
- outimg_legend.png (highlighted-biomarker legend)

## How to apply

Run export2graphlan.py on the BIOM abundance matrix with the --internal_levels flag so that abundances are summed from leaf to root on internal nodes; this emits the GraPhlAn tree file tree.txt and the annotation file annot.txt. Note that export2graphlan requires pandas (ver. 0.13.1), BIOM (ver. 2.0.1) for BIOM inputs, and SciPy (required by hclust2). Annotate the tree with graphlan_annotate.py to produce outtree.txt, then render it with GraPhlAn (which uses the matplotlib library) to draw the circular taxonomy tree as outimg.png along with the annotation-legend image outimg_annot.png and the highlighted-biomarker legend outimg_legend.png. Because export2graphlan accepts BIOM versions 1 and 2, outputs from frameworks such as QIIME and mothur can be fed in directly without tabular conversion.

## Related tools

- **export2graphlan** (Converts BIOM abundance input into the GraPhlAn tree.txt and annot.txt files, summing leaf-to-root abundances with --internal_levels) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Annotates (graphlan_annotate.py) and renders the circular taxonomy tree and legend images via matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Rendering library used by GraPhlAn to draw the circular tree and legend figures)
- **BIOM** (biom-format library (ver. 2.0.1) required by export2graphlan only when input files are in BIOM format) — http://biom-format.org

## Examples

```
export2graphlan.py -i vaginal.biom -t tree.txt -a annot.txt --internal_levels && graphlan_annotate.py annot.txt tree.txt --annot outtree.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- The run produces exactly six expected files: annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt, tree.txt
- outimg.png displays a circular taxonomy tree consistent with the Fig. S3 temporal-dynamics representation
- tree.txt contains internal-node abundance values reflecting leaf-to-root summation (i.e., --internal_levels was honored)
- annot.txt and outimg_legend.png correctly mark biomarker clades above the abundance threshold (default 20.0)

## Limitations

- export2graphlan ver. 0.2.1 requires specific dependency versions: pandas 0.13.1, BIOM 2.0.1 (only for BIOM inputs), and SciPy
- No software version numbers are stated for GraPhlAn itself in the article; the README documents export2graphlan ver. 0.2.1 (27 October 2018)
- Only BIOM versions 1 and 2 are supported; other abundance-table formats must be converted first

## Evidence

- [other] Accepting the widely adopted BIOM format: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [methods] internal levels summation: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] annotation workflow: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] rendering library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] annotation threshold: "Set the minimun abundace value for a clade to be annotated. Default is 20.0"
