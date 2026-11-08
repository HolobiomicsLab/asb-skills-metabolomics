---
name: kegg-module-annotation-mapping
description: Use when a large phylogeny (thousands of taxa, e.g. PhyloXML) must be
  annotated per leaf with KEGG module presence/absence or genome-size metadata as
  external rings in a circular cladogram.
when_to_use_negative:
- Input is a taxonomic abundance profile (e.g. MetaPhlAn2 output) rather than a per-genome/leaf
  phylogeny — use the export2graphlan biomarker workflow instead
- Interactive exploration of large phylogenies is the goal — iTOL targets interactive
  analyses of large-scale phylogenies with overlaid metadata
- Hierarchical quantitative metagenomic data without a reference tree — Krona emphasizes
  hierarchical quantitative information for such profiles
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3053
tools:
- name: GraPhlAn
  role: Renders the annotated circular cladogram with external KEGG module rings and
    genome-size bar plot using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to render the figure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:4e295ce18819992314f52b7f8b4996a13edb1b52e5ed925851bfadca1a79e3d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraPhlAn
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# kegg-module-annotation-mapping

## Summary

Maps per-genome KEGG functional module presence (e.g. M00082, M00083, M00086, M00087, M00088, M00157, M00159) and genome size onto a large phylogenetic tree as GraPhlAn external ring annotations, producing a compact circular cladogram with genome-specific metadata.

## When to use

Use when a large phylogeny (thousands of taxa, e.g. PhyloXML) must be annotated per leaf with KEGG module presence/absence or genome-size metadata as external rings in a circular cladogram.

## When NOT to use

- Input is a taxonomic abundance profile (e.g. MetaPhlAn2 output) rather than a per-genome/leaf phylogeny — use the export2graphlan biomarker workflow instead
- Interactive exploration of large phylogenies is the goal — iTOL targets interactive analyses of large-scale phylogenies with overlaid metadata
- Hierarchical quantitative metagenomic data without a reference tree — Krona emphasizes hierarchical quantitative information for such profiles

## Inputs

- PhyloXML-format phylogenetic tree (e.g. 3,737-taxon PhyloPhlAn tree of life)
- Per-genome KEGG module presence table for modules M00082, M00083, M00086, M00087, M00088, M00157, M00159
- Per-genome genome size values

## Outputs

- Annotated tree file produced by graphlan_annotate.py
- High-quality circular cladogram (Fig. 2-equivalent) with seven KEGG module rings plus a genome-size external circular bar plot, rendered via matplotlib

## How to apply

Load the phylogeny in PhyloXML format (e.g. the 3,737-taxon PhyloPhlAn tree of life from the examples folder of the GraPhlAn repository). Prepare ring annotation data mapping each genome's KEGG module presence — for modules M00082, M00083, M00086, M00087, M00088, M00157, and M00159 retrieved from the KEGG database — and genome size onto its leaf node. Generate the ring annotation commands with graphlan_annotate.py to produce an annotated tree file, then render with GraPhlAn (which uses matplotlib) to obtain a compact circular figure equivalent to the published Fig. 2. Rationale: GraPhlAn was designed to compactly visualize phylogenies spanning thousands of taxa with metadata ranging from microbial community abundances to microbial physiology.

## Related tools

- **GraPhlAn** (Renders the annotated circular cladogram with external KEGG module rings and genome-size bar plot using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to render the figure)

## Examples

```
graphlan_annotate.py --annot kegg_modules.txt tree_of_life.xml annotated_tree.xml && graphlan.py annotated_tree.xml fig2.png
```

## Evaluation signals

- The output figure renders without error
- The figure contains the full 3,737-leaf tree
- Exactly seven KEGG module rings (M00082, M00083, M00086, M00087, M00088, M00157, M00159) are displayed
- A genome-size external circular bar plot ring is present, matching the reference Fig. 2 layout

## Limitations

- No software version numbers are stated for GraPhlAn or its dependencies (matplotlib, pandas, BIOM), hindering exact reproducibility
- Displaying phylogenies with thousands of taxa annotated with metadata is infeasible with most available tools, so alternatives may fail at this scale

## Evidence

- [discussion] KEGG modules: "From the KEGG database (Kanehisa & Goto, 2000; Kanehisa et al., 2014) we focused on the following modules: M00082, M00083, M00086, M00087, M00088, M00157, and M00159."
- [methods] annotation workflow: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] rendering library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [discussion] example data location: "available in the guide folder, inside the examples directory of the GraPhlAn repository (https://bitbucket.org/nsegata/graphlan)"
- [abstract] scope of visualization: "This includes phylogenies spanning up to thousands of taxa, annotated with metadata ranging from microbial community abundances to microbial physiology or host and environmental phenotypes."
