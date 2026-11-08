---
name: graphlan-circular-cladogram-rendering
description: Use when you need to visualize a phylogeny with thousands of taxa together
  with per-leaf or per-clade metadata (abundances, functional modules, phenotypes)
  as a compact circular figure.
when_to_use_negative:
- If the goal is interactive exploration of a large phylogeny with moderate overlaid
  metadata — tools like iTOL target that use case, whereas GraPhlAn is command-driven
  for pipeline reproducibility.
- If the primary display need is hierarchical quantitative metagenomic composition
  (as Krona emphasizes) rather than a phylogenetic cladogram with metadata rings.
- If the input is already a plain abundance matrix without tree structure — use hclust2
  heatmaps instead of forcing a cladogram.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0092
tools:
- name: GraPhlAn
  role: 'Core renderer: generates the compact circular phylogeny with metadata rings
    via matplotlib; run locally or through Galaxy'
  repo: https://bitbucket.org/nsegata/graphlan
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and
    annotation files before rendering
  repo: https://github.com/SegataLab/export2graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to produce the figure
- name: Galaxy
  role: Optional web-based execution interface for GraPhlAn rendering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:44b689dccaca68d0147ce7d6a1cac6a58b4c18f0daaa107bbeb728a305d066a7
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
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# graphlan-circular-cladogram-rendering

## Summary

Render very large phylogenetic trees (e.g., a 3,737-taxon PhyloPhlAn tree of life) as compact, publication-quality circular cladograms with external metadata rings (e.g., KEGG module presence and genome size bars) using GraPhlAn, a command-driven tool built on matplotlib.

## When to use

Use when you need to visualize a phylogeny with thousands of taxa together with per-leaf or per-clade metadata (abundances, functional modules, phenotypes) as a compact circular figure.

## When NOT to use

- If the goal is interactive exploration of a large phylogeny with moderate overlaid metadata — tools like iTOL target that use case, whereas GraPhlAn is command-driven for pipeline reproducibility.
- If the primary display need is hierarchical quantitative metagenomic composition (as Krona emphasizes) rather than a phylogenetic cladogram with metadata rings.
- If the input is already a plain abundance matrix without tree structure — use hclust2 heatmaps instead of forcing a cladogram.

## Inputs

- Phylogenetic tree in PhyloXML format (e.g., 3,737-taxon PhyloPhlAn tree of life)
- Ring annotation data mapping KEGG modules (M00082, M00083, M00086, M00087, M00088, M00157, M00159) and genome size onto leaf nodes
- MetaPhlAn2, LEfSe, or HUMAnN output tables (convertible via export2graphlan)

## Outputs

- High-quality, compact circular cladogram image (e.g., Fig. 2-equivalent Tree of Life with seven KEGG module rings plus genome-size ring)
- Annotated tree file produced by graphlan_annotate.py

## How to apply

Load the phylogeny in PhyloXML format (e.g., the 3,737-taxon PhyloPhlAn tree in the examples folder of the GraPhlAn repository at bitbucket.org/nsegata/graphlan). Prepare ring annotation data mapping metadata such as the seven KEGG modules M00082, M00083, M00086, M00087, M00088, M00157, and M00159 and genome size onto leaf nodes. Generate the annotation commands with graphlan_annotate.py to attach rings and clade highlights to the tree file, then render the circular cladogram with graphlan.py, which uses the matplotlib library. If input comes from MetaPhlAn, LEfSe, or HUMAnN output, convert it first with export2graphlan (ver. 0.2.1) to produce the tree and annotation files, optionally using --background_clades and --background_colors to shade clades and --annotations to control which levels are labeled. Validate that the figure renders without error, contains the full leaf set, and shows the expected number of rings (e.g., seven KEGG module rings plus a genome-size ring) matching the reference layout.

## Related tools

- **GraPhlAn** (Core renderer: generates the compact circular phylogeny with metadata rings via matplotlib; run locally or through Galaxy) — https://bitbucket.org/nsegata/graphlan
- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and annotation files before rendering) — https://github.com/SegataLab/export2graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to produce the figure)
- **Galaxy** (Optional web-based execution interface for GraPhlAn rendering)

## Examples

```
graphlan_annotate.py tree.xml annotations.txt --annot annotated_tree.xml && graphlan.py annotated_tree.xml fig2.png
```

## Evaluation signals

- The output figure renders without error via graphlan.py.
- The rendered cladogram contains the full 3,737-leaf tree from the input PhyloXML file.
- Seven KEGG module rings (M00082, M00083, M00086, M00087, M00088, M00157, M00159) plus the genome-size external circular bar ring are present and match the reference Fig. 2 layout.
- When using export2graphlan, the two output files (tree and annotation) are produced and graphlan_annotate.py successfully attaches the annotations to the tree.
- The figure is publication-quality (vector output via matplotlib) and reproduces with the same command-line inputs.

## Limitations

- No software version numbers were stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM) in the article; only export2graphlan ver. 0.2.1 (27 October 2018) is documented in the README.
- No changelog was found in the repository documentation.
- export2graphlan requires additional libraries: pandas ver. 0.13.1, BIOM ver. 2.0.1 (only for BIOM-format inputs), and SciPy.
- Displaying phylogenies with thousands of microbial taxa in hundreds of samples is infeasible with most available tools; GraPhlAn addresses this but interactivity is limited compared to iTOL/Krona.

## Evidence

- [results] GraPhlAn rendered the 3,737-taxon microbial tree of life inferred by PhyloPhlAn, annotated with seven KEGG functional modules: [evidence span withheld — anchor status: not_found]
- [discussion] GraPhlAn uses the matplotlib library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] outtree.txt is the annotated tree produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [readme] conversion software tool for producing both annotation and tree file for GraPhlAn: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [discussion] available in the guide folder, inside the examples directory of the GraPhlAn repository: "available in the guide folder, inside the examples directory of the GraPhlAn repository (https://bitbucket.org/nsegata/graphlan)"
