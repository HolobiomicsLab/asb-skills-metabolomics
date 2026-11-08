---
name: metadata-ring-annotation
description: Use when you have a large phylogeny (thousands of leaf taxa, e.g., a
  3,737-taxon PhyloPhlAn tree in PhyloXML format) plus per-leaf genome metadata (module
  presence/absence, continuous values like genome size) that must be overlaid as external
  rings on a circular cladogram.
when_to_use_negative:
- Your goal is interactive exploration of a moderate-size tree with metadata overlays
  — iTOL or Krona target that interactive use case; GraPhlAn ring annotation is command-driven
  and aimed at static publication figures
- Your input is a taxonomic/metagenomic abundance table rather than a phylogeny —
  convert profile data via export2graphlan/hclust2 workflows first, or use a heatmap
  tool like hclust2 instead of tree ring annotation
- Your metadata must be attached to internal clade highlights, backgrounds, or biomarker
  shading from LEfSe rather than per-leaf external rings
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GraPhlAn
  role: 'Primary tool: attaches ring annotations to the tree via graphlan_annotate.py
    and renders the compact circular cladogram with external rings'
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to render the circular cladogram
    and its external rings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:b131cb0fbdf91a6bb5685b1cf24ba8a68e6435a5318c1c43c810f69a0c42b4b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraPhlAn
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metadata-ring-annotation

## Summary

Annotate a large phylogenetic tree with genome-specific metadata (e.g., KEGG functional modules and genome size) as concentric external rings using GraPhlAn's annotation mechanism, rendering a compact circular cladogram with matplotlib. This enables visualizing thousands of taxa together with per-genome functional or phenotypic metadata in a single publication-quality figure.

## When to use

Use when you have a large phylogeny (thousands of leaf taxa, e.g., a 3,737-taxon PhyloPhlAn tree in PhyloXML format) plus per-leaf genome metadata (module presence/absence, continuous values like genome size) that must be overlaid as external rings on a circular cladogram.

## When NOT to use

- Your goal is interactive exploration of a moderate-size tree with metadata overlays — iTOL or Krona target that interactive use case; GraPhlAn ring annotation is command-driven and aimed at static publication figures
- Your input is a taxonomic/metagenomic abundance table rather than a phylogeny — convert profile data via export2graphlan/hclust2 workflows first, or use a heatmap tool like hclust2 instead of tree ring annotation
- Your metadata must be attached to internal clade highlights, backgrounds, or biomarker shading from LEfSe rather than per-leaf external rings

## Inputs

- Phylogenetic tree in PhyloXML format (e.g., 3,737-taxon PhyloPhlAn microbial tree of life)
- Tabular per-leaf metadata mapping genome identifiers to KEGG module presence (M00082, M00083, M00086, M00087, M00088, M00157, M00159) and genome size values
- GraPhlAn ring annotation definitions (ring radius, color, and value-to-height mapping per external ring)

## Outputs

- Annotated tree file (outtree.txt) produced by graphlan_annotate.py with the ring annotations attached
- High-quality circular cladogram image (rendered via matplotlib) showing the full tree plus seven KEGG module rings and one genome-size external bar ring, equivalent to the published Fig. 2

## How to apply

Load the phylogeny in PhyloXML format (e.g., the 3,737-taxon PhyloPhlAn tree shipped in the GraPhlAn repository's examples folder at bitbucket.org/nsegata/graphlan). Prepare ring annotation data mapping each genome's metadata onto its leaf node — for the published example, presence of the KEGG modules M00082, M00083, M00086, M00087, M00088, M00157, and M00159 plus genome size, with genome size drawn as an external circular bar plot. Generate the annotation file describing each external ring and run graphlan_annotate.py to attach the annotations to the tree, producing an annotated tree file (outtree.txt). Finally render the annotated tree with GraPhlAn, which is built on matplotlib, to obtain the compact circular cladogram. The decision point is ring design: discrete module-presence values map to solid ring segments/colors, while continuous values such as genome size map to bar heights in the outermost ring.

## Related tools

- **GraPhlAn** (Primary tool: attaches ring annotations to the tree via graphlan_annotate.py and renders the compact circular cladogram with external rings) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to render the circular cladogram and its external rings)

## Examples

```
graphlan_annotate.py tree.xml annotations.txt outtree.txt && graphlan.py outtree.txt fig2.png
```

## Evaluation signals

- The figure renders without errors and contains the full 3,737-leaf tree (leaf count preserved from the input PhyloXML)
- The output displays exactly seven KEGG module rings (M00082, M00083, M00086, M00087, M00088, M00157, M00159) plus one genome-size external bar ring
- Ring segment presence/absence and genome-size bar heights match the per-genome metadata mapping on the corresponding leaf nodes
- The overall layout is comparable to the published reference Fig. 2 in ring ordering and compactness
- The annotated tree file (outtree.txt) is generated by graphlan_annotate.py and can be re-rendered reproducibly by GraPhlAn

## Limitations

- GraPhlAn is command-driven and aimed at static publication figures; it targets interactive analyses of large-scale phylogenies less directly than iTOL
- No software version numbers are stated for GraPhlAn or its dependencies (e.g., matplotlib), so exact reproducibility of rendering may vary across installs
- Metadata must be mappable onto leaf nodes/clades of the tree; data not keyed to tree node names cannot be placed as rings

## Evidence

- [discussion] KEGG modules used in Fig. 2: "From the KEGG database (Kanehisa & Goto, 2000; Kanehisa et al., 2014) we focused on the following modules: M00082, M00083, M00086, M00087, M00088, M00157, and M00159."
- [discussion] matplotlib as rendering engine: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] graphlan_annotate.py output role: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [abstract] scale of trees supported with metadata: "This includes phylogenies spanning up to thousands of taxa, annotated with metadata ranging from microbial community abundances to microbial physiology or host and environmental phenotypes."
- [discussion] example data location in GraPhlAn repo: "available in the guide folder, inside the examples directory of the GraPhlAn repository (https://bitbucket.org/nsegata/graphlan)"
