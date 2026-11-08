---
name: phylogenetic-tree-visualization
description: Use when you need to visualize a large phylogeny (up to thousands of
  taxa, e.g., a 3,737-taxon microbial tree of life) together with genome- or clade-level
  metadata such as KEGG module presence or genome size in a single compact figure.
when_to_use_negative:
- Interactive exploration of large phylogenies with overlaid metadata is the goal
  — iTOL targets interactive analyses and may be more appropriate
- The emphasis is hierarchical quantitative metagenomic taxonomic profiles rather
  than a phylogeny — Krona emphasizes hierarchical quantitative information
- The data is a plain abundance/feature matrix without tree structure (e.g., heatmaps
  are better served by hclust2)
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: GraPhlAn
  role: 'Primary rendering tool: produces compact circular phylogenies annotated with
    diverse, high-dimensional metadata; executed via graphlan_annotate.py and graphlan.py'
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to generate the figure
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and
    annotation input files
  repo: https://github.com/SegataLab/export2graphlan
- name: Galaxy
  role: 'Alternative execution route: GraPhlAn can be run through the Galaxy-based
    web interface instead of the command line'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:c53f549eb1aafb7562f80e24cbb9c6cbdf979d23e605a75ddc6aaf5d4727a578
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# phylogenetic-tree-visualization

## Summary

Render very large phylogenetic trees (thousands of taxa) as compact, high-quality circular cladograms with overlaid metadata rings using GraPhlAn, which builds on matplotlib. This skill covers annotating a PhyloXML tree via graphlan_annotate.py and rendering it into publication-quality figures.

## When to use

Use when you need to visualize a large phylogeny (up to thousands of taxa, e.g., a 3,737-taxon microbial tree of life) together with genome- or clade-level metadata such as KEGG module presence or genome size in a single compact figure.

## When NOT to use

- Interactive exploration of large phylogenies with overlaid metadata is the goal — iTOL targets interactive analyses and may be more appropriate
- The emphasis is hierarchical quantitative metagenomic taxonomic profiles rather than a phylogeny — Krona emphasizes hierarchical quantitative information
- The data is a plain abundance/feature matrix without tree structure (e.g., heatmaps are better served by hclust2)

## Inputs

- PhyloXML-format phylogenetic tree (e.g., 3,737-taxon PhyloPhlAn tree of life)
- Ring/external annotation table mapping leaf nodes to metadata (KEGG modules M00082, M00083, M00086, M00087, M00088, M00157, M00159; genome size)
- MetaPhlAn2, LEfSe, or HUMAnN output tables (convertible via export2graphlan)
- GraPhlAn configuration/annotation file with clade styling and ring parameters

## Outputs

- High-quality circular cladogram image (publication-quality figure, e.g., PNG/PDF via matplotlib)
- Annotated tree file produced by graphlan_annotate.py

## How to apply

Load the phylogeny in PhyloXML format (e.g., the 3,737-taxon PhyloPhlAn microbial tree of life from the examples folder of the GraPhlAn repository at bitbucket.org/nsegata/graphlan). Prepare ring annotation data mapping each leaf's metadata — such as the seven KEGG modules M00082, M00083, M00086, M00087, M00088, M00157, M00159, or genome size — onto its node, plus GraPhlAn global and per-clade styling options. Generate the annotated tree file with graphlan_annotate.py, then render the circular cladogram with GraPhlAn (graphlan.py), which internally uses the matplotlib library. The decision point is granularity: GraPhlAn is designed for compact, static, reproducible publication figures, whereas iTOL targets interactive exploration and Krona emphasizes hierarchical quantitative metagenomic information. For converting MetaPhlAn2, LEfSe, or HUMAnN outputs into GraPhlAn input tree/annotation files, use export2graphlan (installed via Bioconda as 'conda install export2graphlan').

## Related tools

- **GraPhlAn** (Primary rendering tool: produces compact circular phylogenies annotated with diverse, high-dimensional metadata; executed via graphlan_annotate.py and graphlan.py) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to generate the figure)
- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and annotation input files) — https://github.com/SegataLab/export2graphlan
- **Galaxy** (Alternative execution route: GraPhlAn can be run through the Galaxy-based web interface instead of the command line)

## Examples

```
graphlan_annotate.py annotation.txt tree.xml annotated_tree.xml && graphlan.py annotated_tree.xml figure.png
```

## Evaluation signals

- Output figure renders without error via graphlan.py
- Rendered tree contains the full leaf set of the input phylogeny (e.g., all 3,737 taxa of the PhyloPhlAn tree of life)
- Figure displays the expected number of external rings (e.g., seven KEGG module rings plus the genome-size ring) matching the reference layout
- The annotated tree file reflects the annotations attached by graphlan_annotate.py before rendering
- Figure is compact and publication-quality, comparable to the published reference figure

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM), so exact reproducibility of versions is not documented
- export2graphlan requires additional libraries (pandas ver. 0.13.1, BIOM ver. 2.0.1 for BIOM input, SciPy) that must be installed beforehand
- GraPhlAn produces static figures; it is not designed for interactive analysis of large-scale phylogenies

## Evidence

- [results] GraPhlAn rendered the 3,737-taxon microbial tree of life inferred by PhyloPhlAn, annotated with seven KEGG functional modules: [evidence span withheld — anchor status: not_found]
- [abstract] GraPhlAn is a computational tool that produces high-quality, compact visualizations: "We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes."
- [intro] iTOL targets interactive analyses of large-scale phylogenies: "iTOL (Letunic & Bork, 2007; Letunic & Bork, 2011) targets interactive analyses of large-scale phylogenies with a moderate amount of overlaid metadata"
- [discussion] GraPhlAn uses the matplotlib library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [discussion] available in the guide folder, inside the examples directory of the GraPhlAn repository: "available in the guide folder, inside the examples directory of the GraPhlAn repository (https://bitbucket.org/nsegata/graphlan)"
- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn: "**export2graphlan** is a conversion software tool for producing both annotation and tree file for GraPhlAn."
