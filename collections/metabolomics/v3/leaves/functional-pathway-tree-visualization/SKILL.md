---
name: functional-pathway-tree-visualization
description: Use when you have a HUMAnN functional-profile matrix (KEGG pathways)
  plus a LEfSe biomarker result contrasting cohorts (e.g., HMP vs MetaHIT) and need
  a compact, publication-quality circular tree highlighting enriched pathways.
when_to_use_negative:
- Input is taxonomic abundance data (e.g., MetaPhlAn2 profiles) rather than HUMAnN
  functional/KEGG pathway profiles — the taxonomic-tree workflow (Fig. 3) applies
  instead
- You have no LEfSe result file, so export2graphlan cannot automatically highlight
  biomarker sub-trees
- An interactive phylogeny browser (e.g., iTOL) or hierarchical quantitative explorer
  (Krona) is required rather than a static publication-quality circular figure
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN output (here the HUMAnN KEGG functional
    profile and LEfSe result) into GraPhlAn tree.txt and annot.txt files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated functional-ontology tree as a high-quality circular
    figure (outimg.png) using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: graphlan_annotate.py
  role: Attaches the annotations derived by export2graphlan to the tree, producing
    outtree.txt
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics backend used by GraPhlAn to produce the rendered tree images
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:f0099e3b5e8dd1d9e32162be2cb11cb5e71d305bb245c66eca2982c0b4932c12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - graphlan_annotate.py
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# functional-pathway-tree-visualization

## Summary

Reproduce functional-ontology circular trees (e.g., Fig. 4 of the GraPhlAn paper) by converting HUMAnN-derived KEGG functional profiles and LEfSe biomarker results into GraPhlAn tree/annotation files with export2graphlan, annotating them with graphlan_annotate.py, and rendering them with GraPhlAn/matplotlib.

## When to use

Use when you have a HUMAnN functional-profile matrix (KEGG pathways) plus a LEfSe biomarker result contrasting cohorts (e.g., HMP vs MetaHIT) and need a compact, publication-quality circular tree highlighting enriched pathways.

## When NOT to use

- Input is taxonomic abundance data (e.g., MetaPhlAn2 profiles) rather than HUMAnN functional/KEGG pathway profiles — the taxonomic-tree workflow (Fig. 3) applies instead
- You have no LEfSe result file, so export2graphlan cannot automatically highlight biomarker sub-trees
- An interactive phylogeny browser (e.g., iTOL) or hierarchical quantitative explorer (Krona) is required rather than a static publication-quality circular figure

## Inputs

- HUMAnN-derived LEfSe input matrix of KEGG functional profiles (lefse_input)
- LEfSe biomarker result file (lefse_output)
- PIPELINE.sh example script from the export2graphlan/GraPhlAn examples folder

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (rendered circular functional-ontology tree)
- outimg_annot.png and outimg_legend.png (annotated render and legend)

## How to apply

Enter the example folder (e.g., hmp_metahit_metabolic) holding the HUMAnN-derived LEfSe input matrix (lefse_input) and the LEfSe result file (lefse_output). Run ./PIPELINE.sh, which calls export2graphlan.py with -i lefse_input and -o lefse_output to emit tree.txt and annot.txt; optionally use --internal_levels to sum abundances from leaf to root on internal nodes. Annotate the exported tree with graphlan_annotate.py to produce outtree.txt, then invoke GraPhlAn (backed by matplotlib) to render the annotated functional tree. Validation: all six files (annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png) must exist and outimg.png must be a non-empty rendered circular tree; in the reference result, Glycolysis and the Citrate cycle are enriched in HMP while Sulfur Metabolism and Vitamin B6 Metabolism are enriched in MetaHIT.

## Related tools

- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN output (here the HUMAnN KEGG functional profile and LEfSe result) into GraPhlAn tree.txt and annot.txt files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated functional-ontology tree as a high-quality circular figure (outimg.png) using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **graphlan_annotate.py** (Attaches the annotations derived by export2graphlan to the tree, producing outtree.txt) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics backend used by GraPhlAn to produce the rendered tree images)

## Examples

```
./PIPELINE.sh  # runs export2graphlan.py -i lefse_input -o lefse_output -t tree.txt -a annot.txt, then graphlan_annotate.py and GraPhlAn
```

## Evaluation signals

- All six expected files exist: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a non-empty rendered circular tree of the functional ontology (openable PNG produced by matplotlib)
- export2graphlan emitted both tree.txt and annot.txt from -i lefse_input and -o lefse_output
- LEfSe-highlighted KEGG pathways appear as annotated/enlarged nodes: Glycolysis and Citrate cycle enriched in HMP; Sulfur Metabolism and Vitamin B6 Metabolism enriched in MetaHIT (Fig. 4)
- outtree.txt reflects annotation entries added by graphlan_annotate.py on top of tree.txt

## Limitations

- Requires pandas ver. 0.13.1, SciPy (via hclust2), and BIOM ver. 2.0.1 only if inputs are in BIOM format; missing dependencies break the conversion
- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies, so exact reproducibility of Fig. 4 may vary across releases
- No changelog found in the repository documentation
- LEfSe biomarker highlighting is the only automatic annotation signal; sub-trees must derive from the provided LEfSe result, otherwise default clade/font sizes apply (def_clade_size 10, min 20, max 200; def_font_size 10)

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [discussion] The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree.: "The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree."
- [methods] Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format"
- [discussion] HUMAnN was run on the same samples of Fig. 3.: "HUMAnN was run on the same samples of Fig. 3."
- [discussion] GraPhlAn uses the matplotlib library (Hunter, 2007).: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
