---
name: kegg-ortholog-hierarchical-rendering
description: Use when you have a HUMAnN-derived KEGG functional profile matrix plus
  a LEfSe biomarker result and need a publication-quality circular functional tree
  highlighting pathways enriched in contrasting sample cohorts.
when_to_use_negative:
- Input is taxonomic abundance data (e.g., MetaPhlAn2 species profiles) rather than
  functional/KEGG profiles — the taxonomic analog is handled by the taxonomic tree
  workflow (Fig. 3), not this functional-ontology rendering.
- You need interactive exploration of large phylogenies with overlaid metadata — iTOL
  or Krona target interactive analyses, whereas GraPhlAn is command-driven and static.
- Input files are not LEfSe/HUMAnN/MetaPhlAn format and cannot be converted by export2graphlan
  (no supported conversion path described in the README).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts the HUMAnN-derived LEfSe input and LEfSe output into GraPhlAn tree.txt
    and annot.txt files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated circular functional-ontology tree images (backed by
    matplotlib)
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics library underlying GraPhlAn's rendering of the circular tree
- name: hclust2
  role: Optional companion heatmap tool for hierarchical clustering of MetaPhlAn2
    abundance profiles in the same example suite
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:c376f45da930e9a1af2e0b236274080ccbb6ee5743ebbbade0b3853a31e3c108
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
yes_rate: 0.8
n_claims: 15
---

# kegg-ortholog-hierarchical-rendering

## Summary

Render a compact circular tree of KEGG functional-ontology profiles (e.g., HUMAnN-derived KEGG metabolic pathways for HMP vs MetaHIT gut metagenomes) by converting LEfSe/HUMAnN outputs to GraPhlAn input files with export2graphlan, then annotating and plotting with GraPhlAn/matplotlib.

## When to use

Use when you have a HUMAnN-derived KEGG functional profile matrix plus a LEfSe biomarker result and need a publication-quality circular functional tree highlighting pathways enriched in contrasting sample cohorts.

## When NOT to use

- Input is taxonomic abundance data (e.g., MetaPhlAn2 species profiles) rather than functional/KEGG profiles — the taxonomic analog is handled by the taxonomic tree workflow (Fig. 3), not this functional-ontology rendering.
- You need interactive exploration of large phylogenies with overlaid metadata — iTOL or Krona target interactive analyses, whereas GraPhlAn is command-driven and static.
- Input files are not LEfSe/HUMAnN/MetaPhlAn format and cannot be converted by export2graphlan (no supported conversion path described in the README).

## Inputs

- HUMAnN-derived KEGG functional profile in LEfSe input format (lefse_input)
- LEfSe biomarker result file (lefse_output)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png, outimg_annot.png, outimg_legend.png (rendered circular functional tree images)

## How to apply

Enter the example folder (e.g., hmp_metahit_metabolic) containing the HUMAnN-derived LEfSe input matrix and the LEfSe result file. Run export2graphlan.py with -i lefse_input and -o lefse_output to emit tree.txt and annot.txt, optionally using --internal_levels to sum abundances from leaf to root on internal nodes. Annotate the exported tree with graphlan_annotate.py to produce outtree.txt, then invoke GraPhlAn (backed by matplotlib) to render the annotated circular functional tree. Validation: confirm all six expected files (annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png) exist and that outimg.png is a non-empty rendered circular tree; biologically, Glycolysis and the Citrate cycle should be enriched in HMP while Sulfur Metabolism and Vitamin B6 Metabolism are enriched in MetaHIT (Fig. 4).

## Related tools

- **export2graphlan** (Converts the HUMAnN-derived LEfSe input and LEfSe output into GraPhlAn tree.txt and annot.txt files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated circular functional-ontology tree images (backed by matplotlib)) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics library underlying GraPhlAn's rendering of the circular tree)
- **hclust2** (Optional companion heatmap tool for hierarchical clustering of MetaPhlAn2 abundance profiles in the same example suite) — https://github.com/SegataLab/hclust2

## Examples

```
./PIPELINE.sh  # runs export2graphlan.py -i lefse_input -o lefse_output -t tree.txt -a annot.txt, then graphlan_annotate.py and GraPhlAn to render outimg.png
```

## Evaluation signals

- All six expected files exist: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png.
- outimg.png is a non-empty rendered circular tree of the KEGG functional ontology.
- Tree highlights KEGG pathways enriched per cohort: Glycolysis and Citrate cycle enriched in HMP; Sulfur Metabolism and Vitamin B6 Metabolism enriched in MetaHIT (reproduces Fig. 4).
- annot.txt contains highlight/annotation directives derived automatically from the LEfSe biomarkers by export2graphlan.
- outtree.txt equals tree.txt plus the annotations attached by graphlan_annotate.py.

## Limitations

- No software version numbers stated for GraPhlAn or matplotlib in the article; export2graphlan README pins pandas ver. 0.13.1, BIOM ver. 2.0.1 (only for BIOM inputs), and SciPy.
- export2graphlan only converts MetaPhlAn, LEfSe, and/or HUMAnN output formats; other input schemas require prior conversion.
- Annotation depends on LEfSe biomarker detection with an abundance threshold (default 20.0) for a clade to be annotated, so low-abundance pathways may be omitted.
- No changelog found in the repository documentation.

## Evidence

- [discussion] The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree.: "The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree."
- [discussion] HUMAnN was run on the same samples of Fig. 3.: "HUMAnN was run on the same samples of Fig. 3."
- [methods] The ``examples`` folder contains the following sub-folders: ``hmp_aerobiosis``, ``hmp_metahit_metabolic``, and ``hmp_metahit_mp2``.: "The ``examples`` folder contains the following sub-folders: ``hmp_aerobiosis``, ``hmp_metahit_metabolic``, and ``hmp_metahit_mp2``."
- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "**export2graphlan** is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] export2graphlan.py (ver. 0.2.1 of 27 October 2018). Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format.: "export2graphlan.py (ver. 0.2.1 of 27 October 2018). Convert MetaPhlAn, LEfSe,
and/or HUMAnN output to GraPhlAn input format."
