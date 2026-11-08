---
name: chemical-class-propagation-onto-networks
description: Use when a GNPS molecular network plus an MS2LDA experiment exist and
  molecular families lack GNPS spectral library matches, so NAP/DEREPLICATOR annotations
  and ClassyFire classes can be propagated onto nodes/edges to reach MSI annotation
  level 3.
when_to_use_negative:
- Input is already a feature table without a molecular network or component index
  — the motif mapping requires network components (network component index) and shared
  motifs between connected nodes.
- No MS2LDA experiment exists and you only need compound classes without substructure
  motifs — use a chemical-class-only mapping rather than the full motif-propagation
  pipeline.
- Molecular families are already adequately annotated by GNPS spectral library matches
  at MSI level 2, making in silico propagation unnecessary.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3336
tools:
- name: MolNetEnhancer
  role: Overall workflow combining molecular networking, MS2LDA, NAP/DEREPLICATOR,
    and ClassyFire outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: pyMolNetEnhancer
  role: Python module executing the Mass2Motif-to-network and chemical-class mapping
    notebooks
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same integration steps
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: GNPS
  role: Platform for creating the classical or feature-based mass spectral molecular
    network from MGF clustered spectra
- name: MS2LDA
  role: LDA experiment providing Mass2Motif substructure-document relations mapped
    onto the network
- name: Network Annotation Propagation
  role: In silico structure annotation of spectral network nodes; output merged into
    the 'matches' object
- name: ClassyFire
  role: Automated chemical classification of InChIKeys derived from SMILES
- name: molconvert
  role: ChemAxon utility converting SMILES to InChIKeys for ClassyFire input
- name: Cytoscape
  role: 'Visualization of the .graphml output: edges colored by ''interaction'', nodes
    by ''TopSharedMotifs'''
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:2775092f43e8e6bcc7333ae398d5af471234156fab6c7c19fca9925c0f903c21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - Cytoscape
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# chemical-class-propagation-onto-networks

## Summary

MolNetEnhancer-style workflow that merges molecular networking (GNPS), MS2LDA Mass2Motifs, in silico structure annotations (NAP/DEREPLICATOR), and ClassyFire chemical classification to propagate compound-class and substructure information across molecular families, raising annotation from single GNPS library hits to MSI level 3 (e.g. 73 Euphorbia diterpenoid features).

## When to use

Use when a GNPS molecular network plus an MS2LDA experiment exist and molecular families lack GNPS spectral library matches, so NAP/DEREPLICATOR annotations and ClassyFire classes can be propagated onto nodes/edges to reach MSI annotation level 3.

## When NOT to use

- Input is already a feature table without a molecular network or component index — the motif mapping requires network components (network component index) and shared motifs between connected nodes.
- No MS2LDA experiment exists and you only need compound classes without substructure motifs — use a chemical-class-only mapping rather than the full motif-propagation pipeline.
- Molecular families are already adequately annotated by GNPS spectral library matches at MSI level 2, making in silico propagation unnecessary.

## Inputs

- MGF clustered spectra downloaded from GNPS
- GNPS molecular networking job output (GNPS job ID)
- MS2LDA experiment motif-document summary (MS2LDA job ID)
- NAP in silico structure annotation results
- DEREPLICATOR in silico annotation results (optional alternative tool)
- GNPS library hits (gnpslib object)

## Outputs

- Enhanced molecular network in GraphML format with chemical classes and Mass2Motif annotations
- Mass2Motifs_Edges_Classical.tsv edges table (CLUSTERID1 / interact / CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv nodes table with TopSharedMotifs per molecular family
- ClassyFire chemical classification of molecular families at MSI annotation level 3

## How to apply

Create a molecular network through GNPS and an LDA experiment on ms2lda.org from the MGF clustered spectra downloaded from GNPS. Run the pyMolNetEnhancer notebook (e.g. Mass2Motifs_2_Network_Classical.ipynb), supplying the GNPS job ID and MS2LDA job ID with user-defined mapping parameters prob=0.01 (minimal probability score), overlap=0.3 (minimal overlap score), and top=5 (most shared motifs per molecular family). Merge the GNPS library hits, NAP, and DEREPLICATOR outputs into the 'matches' object, then use ChemAxon's molconvert to convert SMILES to InChIKeys and obtain chemical classification through ClassyFire. Combine the motif-mapped network and chemical-class dataframe with make_classyfire_graphml and write the enhanced network via nx.write_graphml; export the edges and nodes tables (e.g. Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv) as intermediates. Finally, import the .graphml into Cytoscape, coloring edges by 'interaction' (Discrete Mapping) and nodes by 'TopSharedMotifs'.

## Related tools

- **MolNetEnhancer** (Overall workflow combining molecular networking, MS2LDA, NAP/DEREPLICATOR, and ClassyFire outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **pyMolNetEnhancer** (Python module executing the Mass2Motif-to-network and chemical-class mapping notebooks) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same integration steps) — https://github.com/madeleineernst/RMolNetEnhancer
- **GNPS** (Platform for creating the classical or feature-based mass spectral molecular network from MGF clustered spectra)
- **MS2LDA** (LDA experiment providing Mass2Motif substructure-document relations mapped onto the network)
- **Network Annotation Propagation** (In silico structure annotation of spectral network nodes; output merged into the 'matches' object)
- **ClassyFire** (Automated chemical classification of InChIKeys derived from SMILES)
- **molconvert** (ChemAxon utility converting SMILES to InChIKeys for ClassyFire input)
- **Cytoscape** (Visualization of the .graphml output: edges colored by 'interaction', nodes by 'TopSharedMotifs')

## Examples

```
pip install pyMolNetEnhancer; # then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb with GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5
```

## Evaluation signals

- The output .graphml loads in Cytoscape and the 'interaction' edge attribute and 'TopSharedMotifs' node/edge table attributes are populated.
- Mass2Motifs_Edges_Classical.tsv imports as a network with CLUSTERID1 as Source Node, 'interact' as Interaction Type, and CLUSTERID2 as Target Node.
- Molecular families that previously had only a single GNPS library-matched feature are annotated at MSI level 3 (putatively characterized compound classes) — e.g. two Euphorbia diterpenoid molecular families totaling 73 molecular features carry the Euphorbia-backbone and nicotinoyl-sidechain Mass2Motif annotations.
- Every edge retained in the motif mapping satisfies both prob >= 0.01 and overlap >= 0.3 thresholds.
- All merged matches contain 'Scan' and 'SMILES' columns that convert successfully to InChIKeys via molconvert before ClassyFire classification.

## Limitations

- Depending on the size of the MS2LDA summary file, a server connection timeout may occur; the file must then be downloaded manually from http://ms2lda.org/.
- The probability and overlap thresholds can be set within the ms2lda.org app under the Experimental Options tab — the summary table there already contains filtered motif-document relations using those thresholds, so repeated filtering can interact with results.
- Integration across tools is hampered because each tool has its own output format and efficient matching of data across tools is lacking; MolNetEnhancer bridges this but DEREPLICATOR and pyMolNetEnhancer outputs are complementary, so both should be used for annotation.

## Evidence

- [abstract] workflow combining outputs: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical"
- [methods] molconvert usage: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
- [readme] prob parameter: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [intro] pyMolNetEnhancer definition: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular"
- [readme] MS2LDA timeout note: "Note</b>: Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
