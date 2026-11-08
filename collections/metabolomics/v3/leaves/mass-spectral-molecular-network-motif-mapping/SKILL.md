---
name: mass-spectral-molecular-network-motif-mapping
description: Use when you have GNPS molecular network outputs (graphml, clustered
  MGF, edges/nodes files) plus an MS2LDA experiment and need to link substructure
  motifs to network nodes/edges for annotation and visualization.
when_to_use_negative:
- Input is a feature-based molecular network built with MZmine rather than a classical
  GNPS network — use the Mass2Motifs_2_Network_FeatureBased workflow instead of the
  classical one
- 'No MS2LDA experiment exists yet: motifs must first be discovered on ms2lda.org
  from the GNPS clustered MGF before mapping is possible'
- Goal is purely chemical class annotation without substructure motifs — the chemical-class
  mapping branch of MolNetEnhancer is the appropriate route, not Mass2Motif mapping
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyMolNetEnhancer
  role: Python module that filters Mass2Motif-document relations by prob/overlap thresholds
    and maps MS2LDA substructural information onto classical and feature-based GNPS
    molecular networks; executed via the Mass2Motifs_2_Network_Classical.ipynb notebook
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same motif-to-network mapping workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Discovers Mass2Motifs (substructures) from the GNPS clustered MGF; its motif
    summary table is the direct input to the mapping step
  repo: http://ms2lda.org/
- name: GNPS
  role: Platform on which the classical molecular network (graphml, clustered MGF,
    edges/nodes files) is created before motif mapping
  repo: https://gnps.ucsd.edu/
- name: DEREPLICATOR
  role: In silico structure annotation used to annotate the mapped Mass2Motifs and
    link motifs to known peptidic structures; complementary to pyMolNetEnhancer output
- name: Cytoscape
  role: 'Visualization of the .graphml output: edges colored by shared Mass2Motifs
    via the ''interaction'' column (Discrete Mapping), nodes via TopSharedMotifs;
    alternatively import Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv
    separately'
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:a8e375c1c73eb65057c0cf5bedad38fc5588e124a95c2bd8b8f35e10a9cdeca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# mass-spectral-molecular-network-motif-mapping

## Summary

Maps MS2LDA Mass2Motif substructural annotations onto GNPS mass spectral molecular networks using the MolNetEnhancer/pyMolNetEnhancer workflow, enabling discovery of subtle substructural diversity within molecular families such as peptidic motifs in Xenorhabdus/Photorhabdus extracts.

## When to use

Use when you have GNPS molecular network outputs (graphml, clustered MGF, edges/nodes files) plus an MS2LDA experiment and need to link substructure motifs to network nodes/edges for annotation and visualization.

## When NOT to use

- Input is a feature-based molecular network built with MZmine rather than a classical GNPS network — use the Mass2Motifs_2_Network_FeatureBased workflow instead of the classical one
- No MS2LDA experiment exists yet: motifs must first be discovered on ms2lda.org from the GNPS clustered MGF before mapping is possible
- Goal is purely chemical class annotation without substructure motifs — the chemical-class mapping branch of MolNetEnhancer is the appropriate route, not Mass2Motif mapping

## Inputs

- GNPS classical molecular network job outputs (.graphml network, clustered MGF, edges and nodes files) e.g. for dataset MSV000081063
- MS2LDA experiment output from ms2lda.org (motif summary / MotifFS2.csv motifs_from_documents table of Mass2Motif-document relations)
- In silico structure annotation results, e.g. DEREPLICATOR/VarQuest outputs with 'Scan' and 'SMILES' columns
- User-defined mapping parameters: prob, overlap, top

## Outputs

- Motif-annotated .graphml molecular network with Mass2Motif presence per node and edge
- Mass2Motifs_Edges_Classical.tsv edge table (CLUSTERID1, interact, CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv node table
- TopSharedMotifs column listing the most shared motifs per molecular family (network component index)

## How to apply

Create a classical molecular network through the GNPS platform, cluster the spectra into an MGF, and run an LDA experiment on ms2lda.org using that clustered MGF; large motif summary exports (e.g., motifs_from_documents tables) may need manual download due to server connection timeouts. Run the Mass2Motifs_2_Network_Classical.ipynb notebook from pyMolNetEnhancer line by line, specifying your GNPS job ID, MS2LDA job ID, and the mapping parameters prob=0.01 (minimal probability score), overlap=0.3 (minimal overlap score), and top=5 (most shared motifs per molecular family/network component). Filter the Mass2Motif-document relations by these thresholds, then map motifs onto nodes and edges via pyMolNetEnhancer (for per-edge motif interactions use EdgesMotifs2Network), computing the TopSharedMotifs column per network component. Optionally annotate the mapped motifs with in silico structure results (DEREPLICATOR/VarQuest outputs with 'Scan' and 'SMILES' columns) to link motifs to known structures. Write the annotated network via networkx to .graphml and export Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv for Cytoscape visualization, coloring edges by the 'interaction' column (Discrete Mapping) and nodes by TopSharedMotifs.

## Related tools

- **pyMolNetEnhancer** (Python module that filters Mass2Motif-document relations by prob/overlap thresholds and maps MS2LDA substructural information onto classical and feature-based GNPS molecular networks; executed via the Mass2Motifs_2_Network_Classical.ipynb notebook) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same motif-to-network mapping workflow) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Discovers Mass2Motifs (substructures) from the GNPS clustered MGF; its motif summary table is the direct input to the mapping step) — http://ms2lda.org/
- **GNPS** (Platform on which the classical molecular network (graphml, clustered MGF, edges/nodes files) is created before motif mapping) — https://gnps.ucsd.edu/
- **DEREPLICATOR** (In silico structure annotation used to annotate the mapped Mass2Motifs and link motifs to known peptidic structures; complementary to pyMolNetEnhancer output)
- **Cytoscape** (Visualization of the .graphml output: edges colored by shared Mass2Motifs via the 'interaction' column (Discrete Mapping), nodes via TopSharedMotifs; alternatively import Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv separately) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer; # then execute Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb line by line with GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5
```

## Evaluation signals

- Reproduced motif frequency counts must match the reported values: lys-lys 1500/6228, ethylphenyl 478/6228, rhabdopeptide 231 and 186, leu-leu 110, xentrivalpeptide 28 out of 6228 network nodes
- Annotation counts must match: 46 annotated motifs, of which 32 are peptidic and 11 linked to known peptides produced by the strains
- For a 32-member xenoamicin molecular family, motif membership must break down exactly as 4 (ring A + tail), 10 (ring A only), 3 (tail only), 2 (ring A + putative ring B + tail), 2 (putative ring B + tail), with overlap score > 0.3
- Ring and tail Mass2Motifs must show fragment consistency with DEREPLICATOR/VarQuest fragment annotations in more than 75% of their linked molecular features
- The exported .graphml must import into Cytoscape so that edges can be colored by 'interaction' (Discrete Mapping) and nodes by TopSharedMotifs from the Edge Table

## Limitations

- Large MS2LDA motif summary files may cause a server connection timeout on ms2lda.org, requiring manual download of the summary via the Export MS2LDA Summary option
- The MS2LDA summary table already contains filtered motif-document relations using thresholds set in the ms2lda.org web app under Experimental Options, so thresholds must be consistent between the web app and the mapping script
- pyMolNetEnhancer and DEREPLICATOR outputs are complementary — neither alone fully annotates a molecular network, so both should be used

## Evidence

- [abstract] workflow combines molecular networking, MS2LDA and in silico annotation: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR)"
- [methods] filter thresholds prob and overlap: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [methods] top parameter for most shared motifs per family: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [methods] create LDA experiment from GNPS clustered MGF: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
- [methods] install pyMolNetEnhancer: "Install pyMolNetEnhancer with:

 `pip install pyMolNetEnhancer`"
- [readme] summary table already filtered in web app: "the summary table contains filtered motif-document relations using the set thresholds in the web app"
- [readme] visualize graphml in Cytoscape: "To visualize results import the .graphml output file into [Cytoscape](https://cytoscape.org/)."
- [readme] server timeout for MS2LDA summary download: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
