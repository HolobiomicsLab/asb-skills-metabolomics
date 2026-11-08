---
name: humann-output-conversion
description: Use when you have a HUMAnN functional-profile matrix (e.g., KEGG pathways)
  plus a LEfSe result file and need to render them as an annotated GraPhlAn circular
  functional tree.
when_to_use_negative:
- Input is a taxonomic abundance profile you want as a heat-map — use hclust2 instead
  of a functional-tree conversion
- You need interactive exploration of large phylogenies rather than a static publication
  figure — iTOL or Krona may be more appropriate
- Input files are not MetaPhlAn/LEfSe/HUMAnN outputs and lack the hierarchical pathway
  naming export2graphlan expects
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts the HUMAnN/LEfSe files into GraPhlAn tree.txt and annot.txt; invoked
    with -i lefse_input and -o lefse_output
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated functional-ontology tree into circular PNG images after
    graphlan_annotate.py
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics library used by GraPhlAn to produce the final rendered figures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:8bb1d84c8d73fe514466ad0556eecbc5fba3caeaad1f698994df429f4f924b7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# humann-output-conversion

## Summary

Convert HUMAnN-derived KEGG functional profiles and LEfSe biomarker results into GraPhlAn tree and annotation files using export2graphlan, enabling circular visualization of functional-ontology contrasts such as the HMP vs MetaHIT gut metagenome tree (Fig. 4).

## When to use

Use when you have a HUMAnN functional-profile matrix (e.g., KEGG pathways) plus a LEfSe result file and need to render them as an annotated GraPhlAn circular functional tree.

## When NOT to use

- Input is a taxonomic abundance profile you want as a heat-map — use hclust2 instead of a functional-tree conversion
- You need interactive exploration of large phylogenies rather than a static publication figure — iTOL or Krona may be more appropriate
- Input files are not MetaPhlAn/LEfSe/HUMAnN outputs and lack the hierarchical pathway naming export2graphlan expects

## Inputs

- HUMAnN-derived LEfSe input matrix of KEGG functional profiles (e.g., lefse_input from hmp_metahit_metabolic)
- LEfSe biomarker result file (e.g., lefse_output)
- Optional BIOM-format input table (requires biom-format 2.0.1; pandas 0.13.1 and SciPy also required)

## Outputs

- tree.txt (GraPhlAn tree file of the functional ontology)
- annot.txt (GraPhlAn annotation file highlighting LEfSe biomarkers)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png, outimg_annot.png, outimg_legend.png (rendered circular functional tree images)

## How to apply

Run export2graphlan.py with -i pointing to the LEfSe input matrix (the HUMAnN functional profile, e.g. lefse_input) and -o pointing to the LEfSe result (e.g. lefse_output), emitting tree.txt and annot.txt; optionally pass --internal_levels to sum abundances from leaf to root on internal nodes of the functional ontology. Next, attach annotations with graphlan_annotate.py to produce outtree.txt, then render with GraPhlAn (matplotlib-backed) to obtain outimg.png, outimg_annot.png, and outimg_legend.png. This mirrors the hmp_metahit_metabolic example, where HUMAnN was run with KEGG ortholog groups and LEfSe identified biomarker pathways before conversion. The rationale is that export2graphlan automatically highlights LEfSe biomarker sub-trees, avoiding manual annotation of the functional hierarchy.

## Related tools

- **export2graphlan** (Converts the HUMAnN/LEfSe files into GraPhlAn tree.txt and annot.txt; invoked with -i lefse_input and -o lefse_output) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated functional-ontology tree into circular PNG images after graphlan_annotate.py) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics library used by GraPhlAn to produce the final rendered figures)

## Examples

```
./PIPELINE.sh  # invokes: export2graphlan.py -i lefse_input -o lefse_output -t tree.txt -a annot.txt; graphlan_annotate.py tree.txt annot.txt outtree.txt; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six expected files exist: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a non-empty rendered circular tree of the functional ontology
- LEfSe biomarker pathways (e.g., Glycolysis and Citrate cycle enriched in HMP; Sulfur Metabolism and Vitamin B6 Metabolism enriched in MetaHIT) appear highlighted in the annotation file and legend
- tree.txt nodes reflect the KEGG pathway hierarchy from the HUMAnN functional profile, with --internal_levels abundances summed leaf-to-root if that flag was used
- The rendered figure reproduces the contrast expected from the LEfSe result (Fig. 4 topology)

## Limitations

- export2graphlan requires specific dependency versions (pandas 0.13.1, BIOM 2.0.1 for BIOM inputs, SciPy for hclust2); no version numbers are stated for GraPhlAn itself
- No changelog is present in the repository documentation, complicating reproducibility pinning
- Biomarker highlighting derives automatically from the LEfSe result; if LEfSe output is absent or empty, annot.txt will not mark any sub-trees

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [discussion] HUMAnN was run on the same samples of Fig. 3 with KEGG ortholog groups: "HUMAnN was run on the same samples of Fig. 3."
- [methods] outtree.txt is the annotated tree produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [methods] internal_levels sums abundances leaf to root: "--internal_levels     If specified sum-up from leaf to root the abundances values."
