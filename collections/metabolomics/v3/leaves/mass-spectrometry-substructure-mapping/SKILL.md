---
name: mass-spectrometry-substructure-mapping
description: Use when a GNPS molecular network (classical or feature-based) already
  exists and MS2LDA motif-document probability/overlap relations must be attached
  to nodes and edges for substructural visualization.
when_to_use_negative:
- The molecular network itself has not yet been created (no GNPS job output available)
  — create the network through GNPS first, then apply this skill.
- The LDA experiment has not been run on ms2lda.org using the MGF clustered spectra
  (or MZmine MGF) — no Mass2Motif relations exist to map.
- The goal is chemical class annotation rather than MS2LDA substructure mapping —
  use the ChemicalClasses_to_Network workflow instead.
edam_operation: http://edamontology.org/operation_0515
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Python module that executes the Mass2Motif-to-network mapping; accepts GNPS
    job ID, MS2LDA job ID/summary, and prob/overlap/top thresholds; exports .graphml
    and TSV outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same Mass2Motif-to-network mapping workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Generates the Mass2Motifs (motif-document probability and overlap relations)
    that are mapped onto the network; run on ms2lda.org
- name: GNPS
  role: Platform that produces the mass spectral molecular network (classical or feature-based)
    onto which the motifs are mapped
- name: Cytoscape
  role: 'Visualization of the mapped motifs: edges colored by ''interaction'' (Discrete
    Mapping) and nodes by ''TopSharedMotifs'' (Image/Chart)'
- name: networkx
  role: Writes the annotated network to Motif_ChemicalClass_Network_Classical.graphml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:f5f615af9f9f3bf96284fed51ec8b41fcaf0d5c494243aaf5214b6e70f39eabf
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
  edam_operation: http://edamontology.org/operation_0515
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mass-spectrometry-substructure-mapping

## Summary

Maps MS2LDA-discovered Mass2Motifs onto GNPS mass spectral molecular networks so that each node carries a pie-chart of its contained motifs and each edge is annotated with the motifs shared between nodes.

## When to use

Use when a GNPS molecular network (classical or feature-based) already exists and MS2LDA motif-document probability/overlap relations must be attached to nodes and edges for substructural visualization.

## When NOT to use

- The molecular network itself has not yet been created (no GNPS job output available) — create the network through GNPS first, then apply this skill.
- The LDA experiment has not been run on ms2lda.org using the MGF clustered spectra (or MZmine MGF) — no Mass2Motif relations exist to map.
- The goal is chemical class annotation rather than MS2LDA substructure mapping — use the ChemicalClasses_to_Network workflow instead.

## Inputs

- GNPS molecular networking job output (classical network or feature-based network)
- MS2LDA job ID or manually downloaded MS2LDA summary file (motif-document probability and overlap relations)
- MGF clustered spectra downloaded from GNPS (classical) or MGF file created within MZmine (feature-based) used to build the ms2lda.org LDA experiment

## Outputs

- Motif_ChemicalClass_Network_Classical.graphml annotated network file for import into Cytoscape
- Mass2Motifs_Edges_Classical.tsv (edge list with CLUSTERID1, interact, CLUSTERID2 columns)
- Mass2Motifs_Nodes_Classical.tsv (node table with motif annotations)
- Node attribute TopSharedMotifs listing up to 'top' most shared Mass2Motifs per molecular family
- Edge attribute 'interaction' encoding shared Mass2Motifs between nodes

## How to apply

Install pyMolNetEnhancer (pip install pyMolNetEnhancer) and run the Mass2Motifs_2_Network_Classical (or _FeatureBased) notebook, supplying the GNPS job ID, the MS2LDA job ID or a manually downloaded MS2LDA summary file, and three mapping thresholds: prob = 0.01 (minimal motif-document probability), overlap = 0.3 (minimal overlap score), and top = 5 (number of most shared motifs shown per molecular family / network component index). The probability and overlap thresholds can equivalently be set in the ms2lda.org web app under the Experimental Options tab; the summary table then contains filtered motif-document relations using those thresholds. The mapping annotates each node with its contained Mass2Motifs (via the TopSharedMotifs attribute) and adds motif-sharing edge attributes (interaction), exporting Motif_ChemicalClass_Network_Classical.graphml together with Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv. In Cytoscape, color edges by 'interaction' (Stroke Color, Discrete Mapping) and nodes by the Mass2Motifs in 'TopSharedMotifs' (Image/Chart) to verify the mapping visually. For feature-based networks, the LDA experiment is built from the MGF file created within MZmine; for classical networks, from the MGF clustered spectra downloaded from GNPS.

## Related tools

- **pyMolNetEnhancer** (Python module that executes the Mass2Motif-to-network mapping; accepts GNPS job ID, MS2LDA job ID/summary, and prob/overlap/top thresholds; exports .graphml and TSV outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same Mass2Motif-to-network mapping workflow) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Generates the Mass2Motifs (motif-document probability and overlap relations) that are mapped onto the network; run on ms2lda.org)
- **GNPS** (Platform that produces the mass spectral molecular network (classical or feature-based) onto which the motifs are mapped)
- **Cytoscape** (Visualization of the mapped motifs: edges colored by 'interaction' (Discrete Mapping) and nodes by 'TopSharedMotifs' (Image/Chart))
- **networkx** (Writes the annotated network to Motif_ChemicalClass_Network_Classical.graphml)

## Examples

```
pip install pyMolNetEnhancer; then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb line by line (specify GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5)
```

## Evaluation signals

- The exported .graphml file loads in Cytoscape and every node carries a TopSharedMotifs value listing at most 5 motifs (matching the 'top' parameter).
- Edge attribute 'interaction' is populated with shared Mass2Motifs and edge Stroke Color can be assigned via Discrete Mapping in Cytoscape.
- Filtered motif-document relations respect the thresholds: only relations with probability ≥ 0.01 and overlap ≥ 0.3 (or the user-set values) are present in the summary used for mapping.
- The Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv files import correctly into Cytoscape (Source Node = CLUSTERID1, Interaction = interact, Target Node = CLUSTERID2).
- Node fill colors in Cytoscape can be mapped discretely by shared Mass2Motifs, reproducing the pie-chart motif representation on network nodes.

## Limitations

- Depending on the size of the MS2LDA summary file, a server connection timeout may occur; in that case the file must be downloaded manually from http://ms2lda.org/ (Export MS2LDA Summary).
- The probability and overlap thresholds should ideally be set within the ms2lda.org app under the Experimental Options tab, because the summary table only contains motif-document relations filtered by the thresholds set there.
- Prob, overlap, and top thresholds are user-defined and influence which motifs appear on nodes and edges; changing them changes the visual annotation.
- For feature-based networks the LDA experiment must be built from the MGF file created within MZmine, not from the classical GNPS clustered MGF.

## Evidence

- [readme] prob: minimal probability score for a Mass2Motif to be included. Default is 0.01.: "prob: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [readme] overlap and top thresholds define which motifs appear per molecular family: [evidence span withheld — anchor status: not_found]
- [readme] MS2LDA summary may need manual download due to server timeout: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
- [readme] Cytoscape node coloring via TopSharedMotifs: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left and select Mass2Motifs shown in 'TopSharedMotifs' in"
- [methods] MS2LDA experiment must use the MGF clustered spectra downloaded from GNPS: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
