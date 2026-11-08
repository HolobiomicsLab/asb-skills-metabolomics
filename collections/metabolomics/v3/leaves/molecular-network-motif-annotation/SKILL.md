---
name: molecular-network-motif-annotation
description: Use when you have both a GNPS molecular networking job output and an
  MS2LDA experiment summary (motif-document probabilities and overlap relations) and
  need to annotate network nodes and edges with shared Mass2Motifs for Cytoscape visualization.
when_to_use_negative:
- The molecular network was created with the feature-based (FBMN) workflow, e.g. via
  MZmine — use the Mass2Motifs_2_Network_FeatureBased notebook instead of the classical
  one
- The goal is annotating networks with chemical classes (ClassyFire) rather than MS2LDA
  substructure motifs — that is a separate MolNetEnhancer step
- No MS2LDA experiment exists for the spectra; motifs cannot be mapped without motif-document
  probability and overlap relations
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Python module that runs the Mass2Motifs_2_Network_Classical mapping, annotating
    nodes and edges with Mass2Motif information and writing the graphml/TSV outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package providing the same Mass2Motifs-to-network mapping workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Source of the Mass2Motifs; the LDA experiment run on ms2lda.org provides motif-document
    probabilities and overlap scores used as mapping input
- name: GNPS
  role: Platform that produces the molecular network whose nodes are annotated; its
    MGF-clustered spectra feed the MS2LDA experiment
- name: Cytoscape
  role: 'Visualization environment for the .graphml output: node pie charts via ''Image/Chart''
    on TopSharedMotifs and edge stroke color via Discrete Mapping on ''interaction'''
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:eef88d35876ea55d317c6a96b346297f5630c0a905471c9f740f4d4949976cec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# molecular-network-motif-annotation

## Summary

Maps MS2LDA-discovered Mass2Motifs onto nodes of a GNPS molecular network so each node carries a pie-chart of the motifs it contains, with relative abundance reflecting the motif's overlap score. This is MolNetEnhancer's first integration step, implemented in pyMolNetEnhancer or RMolNetEnhancer.

## When to use

Use when you have both a GNPS molecular networking job output and an MS2LDA experiment summary (motif-document probabilities and overlap relations) and need to annotate network nodes and edges with shared Mass2Motifs for Cytoscape visualization.

## When NOT to use

- The molecular network was created with the feature-based (FBMN) workflow, e.g. via MZmine — use the Mass2Motifs_2_Network_FeatureBased notebook instead of the classical one
- The goal is annotating networks with chemical classes (ClassyFire) rather than MS2LDA substructure motifs — that is a separate MolNetEnhancer step
- No MS2LDA experiment exists for the spectra; motifs cannot be mapped without motif-document probability and overlap relations

## Inputs

- GNPS molecular networking job output (classical workflow) or job ID
- MS2LDA job ID or manually downloaded MS2LDA summary file (motif-document probability and overlap relations)
- MGF clustered spectra downloaded from GNPS, used to create the LDA experiment on ms2lda.org

## Outputs

- Motif_ChemicalClass_Network_Classical.graphml (network with TopSharedMotifs node attribute and 'interaction' edge attribute)
- Mass2Motifs_Edges_Classical.tsv (CLUSTERID1, interact, CLUSTERID2 columns for edge import into Cytoscape)
- Mass2Motifs_Nodes_Classical.tsv (node table for separate import into Cytoscape)

## How to apply

Create a molecular network through the GNPS platform, download the MGF-clustered spectra, and run an LDA experiment on ms2lda.org. In pyMolNetEnhancer, run the Mass2Motifs_2_Network_Classical notebook line by line, specifying the GNPS job ID, the MS2LDA job ID (or a manually downloaded MS2LDA summary file if a server timeout occurs), and the mapping parameters: prob = 0.01 (minimal probability score for a Mass2Motif to be included), overlap = 0.3 (minimal overlap score for inclusion), and top = 5 (number of most shared motifs shown per molecular family, i.e., network component index). The mapping annotates each node with its contained Mass2Motifs (TopSharedMotifs attribute, rendered as pie charts) and adds motif-sharing edge attributes ('interaction'). Export the annotated network as a .graphml file via networkx.write_graphml, plus Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv. Note that prob and overlap thresholds can alternatively be set in the ms2lda.org app under Experimental Options, in which case the summary table already contains filtered motif-document relations.

## Related tools

- **pyMolNetEnhancer** (Python module that runs the Mass2Motifs_2_Network_Classical mapping, annotating nodes and edges with Mass2Motif information and writing the graphml/TSV outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package providing the same Mass2Motifs-to-network mapping workflow) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Source of the Mass2Motifs; the LDA experiment run on ms2lda.org provides motif-document probabilities and overlap scores used as mapping input)
- **GNPS** (Platform that produces the molecular network whose nodes are annotated; its MGF-clustered spectra feed the MS2LDA experiment)
- **Cytoscape** (Visualization environment for the .graphml output: node pie charts via 'Image/Chart' on TopSharedMotifs and edge stroke color via Discrete Mapping on 'interaction')

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- The exported .graphml file loads in Cytoscape without errors
- Node fill can be mapped discretely by shared Mass2Motifs via 'Image/Chart' on the TopSharedMotifs attribute, and edge stroke color via 'interaction' with Discrete Mapping
- Only motif-document relations passing prob = 0.01 and overlap = 0.3 thresholds are represented in the mapping
- At most top = 5 most shared motifs per molecular family (network component index) are shown per component
- Separate TSV imports reconstruct the same network: CLUSTERID1 as Source Node, 'interact' as Interaction Type, CLUSTERID2 as Target Node

## Limitations

- Depending on the size of the MS2LDA summary file, a server connection timeout may occur; the file may need to be downloaded manually from ms2lda.org
- The MS2LDA summary table contains motif-document relations already filtered by the thresholds set in the web app's Experimental Options tab, so web-app settings interact with the prob/overlap parameters supplied to pyMolNetEnhancer
- Only the most shared motifs per molecular family (default top = 5) are shown, so less frequently shared motifs within a component may not be visualized

## Evidence

- [abstract] workflow combines molecular networking with MS2LDA: "Here, we introduce MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools"
- [methods] prob threshold definition: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [methods] overlap threshold definition: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] LDA experiment creation from GNPS MGF: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
- [readme] top parameter definition: "This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [readme] Cytoscape node visualization: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left"
- [readme] server timeout fallback: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
