---
name: gnps-ms2lda-job-output-harmonization
description: Use when a GNPS molecular network exists alongside an MS2LDA LDA experiment
  and NAP/DEREPLICATOR results, and these tool outputs with differing formats must
  be merged so Mass2Motifs and chemical classes can be mapped onto molecular families.
when_to_use_negative:
- There is no MS2LDA LDA experiment for the dataset and no motif-document summary
  can be retrieved, so motif-to-network mapping cannot be performed
- A single GNPS spectral library match per molecular family is sufficient and no substructural/chemical-class
  enrichment beyond library hits is needed (e.g. the un-enhanced Euphorbia network
  had only one library-matched feature before MolNetEnhancer raised it to MSI level
  3)
- The goal is DEREPLICATOR-style annotation only, without integrating MS2LDA motifs
  and ClassyFire classes — the paper notes PyMolNetEnhancer and DEREPLICATOR outputs
  are complementary but distinct
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pyMolNetEnhancer
  role: Python module whose Mass2Motifs_2_Network_Classical notebook merges GNPS/MS2LDA/NAP
    outputs and builds the enhanced GraphML network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for integrating chemical class and substructure information
    within GNPS molecular networks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: GNPS
  role: Source of the molecular networking job output and MGF clustered spectra that
    anchor the harmonization
- name: MS2LDA
  role: Provides the motif-document summary (LDA experiment) whose Mass2Motifs are
    mapped onto the network with prob/overlap/top thresholds
- name: Network Annotation Propagation
  role: In silico structure annotation results merged into the 'matches' object alongside
    GNPS library hits
- name: ClassyFire
  role: Automated chemical classification applied to molconvert-derived InChIKeys
    to build the chemical-class dataframe
- name: molconvert
  role: ChemAxon utility converting SMILES from the merged matches table into InChIKeys
    for ClassyFire input
- name: Cytoscape
  role: Visualization of the .graphml output; edges colored by shared Mass2Motifs
    via the 'interaction' column with Discrete Mapping
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:fb801589a64d18e9d36d5c16ba7e0a36248f63373a5019e06f084a3cc5aa415e
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# GNPS-MS2LDA job output harmonization

## Summary

Harmonizes heterogeneous outputs from GNPS molecular networking, MS2LDA motif-document summaries, and in silico annotation tools (NAP/DEREPLICATOR) into a single merged 'matches' object (Scan + SMILES columns), converts SMILES to InChIKeys with ChemAxon's molconvert, and feeds ClassyFire classifications into a pyMolNetEnhancer-enhanced GraphML molecular network.

## When to use

Use when a GNPS molecular network exists alongside an MS2LDA LDA experiment and NAP/DEREPLICATOR results, and these tool outputs with differing formats must be merged so Mass2Motifs and chemical classes can be mapped onto molecular families.

## When NOT to use

- There is no MS2LDA LDA experiment for the dataset and no motif-document summary can be retrieved, so motif-to-network mapping cannot be performed
- A single GNPS spectral library match per molecular family is sufficient and no substructural/chemical-class enrichment beyond library hits is needed (e.g. the un-enhanced Euphorbia network had only one library-matched feature before MolNetEnhancer raised it to MSI level 3)
- The goal is DEREPLICATOR-style annotation only, without integrating MS2LDA motifs and ClassyFire classes — the paper notes PyMolNetEnhancer and DEREPLICATOR outputs are complementary but distinct

## Inputs

- GNPS molecular networking job output (classical or feature-based network job)
- MGF clustered spectra downloaded from GNPS (used to create the ms2lda.org LDA experiment)
- MS2LDA motif-document summary (summary table from ms2lda.org experiment)
- NAP in silico structure annotation results
- Merged 'matches' tab-separated table with 'Scan' and 'SMILES' columns

## Outputs

- Enhanced molecular network as .graphml (from make_classyfire_graphml via nx.write_graphml)
- Mass2Motifs_Edges_Classical.tsv edges table
- Mass2Motifs_Nodes_Classical.tsv nodes table
- InChIKeys derived from SMILES via molconvert
- ClassyFire chemical class annotations mapped onto network nodes

## How to apply

Retrieve the GNPS molecular networking job output and the MS2LDA motif-document summary from ms2lda.org (download manually if a server connection timeout occurs), plus the NAP in silico structure annotation results. Run the pyMolNetEnhancer Mass2Motifs_2_Network_Classical notebook, supplying the GNPS job ID, MS2LDA job ID, and mapping parameters prob=0.01 (minimal probability score), overlap=0.3 (minimal overlap score), and top=5 (most shared motifs per molecular family). Merge the GNPS library hits and NAP output into the 'matches' object (matches = [gnpslib, nap, derep]) as a tab-separated table with 'Scan' and 'SMILES' columns, then use molconvert to convert SMILES to InChIKeys and obtain chemical classification through ClassyFire. Combine the motif-mapped network (MG) with the chemical-class dataframe (final) via make_classyfire_graphml and write the enhanced network with nx.write_graphml. The rationale is that each tool has its own output format, so explicit merging and identifier conversion (SMILES→InChIKey) is required before ClassyFire-level chemistry can be attached to network nodes and edges.

## Related tools

- **pyMolNetEnhancer** (Python module whose Mass2Motifs_2_Network_Classical notebook merges GNPS/MS2LDA/NAP outputs and builds the enhanced GraphML network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for integrating chemical class and substructure information within GNPS molecular networks) — https://github.com/madeleineernst/RMolNetEnhancer
- **GNPS** (Source of the molecular networking job output and MGF clustered spectra that anchor the harmonization)
- **MS2LDA** (Provides the motif-document summary (LDA experiment) whose Mass2Motifs are mapped onto the network with prob/overlap/top thresholds)
- **Network Annotation Propagation** (In silico structure annotation results merged into the 'matches' object alongside GNPS library hits)
- **ClassyFire** (Automated chemical classification applied to molconvert-derived InChIKeys to build the chemical-class dataframe)
- **molconvert** (ChemAxon utility converting SMILES from the merged matches table into InChIKeys for ClassyFire input)
- **Cytoscape** (Visualization of the .graphml output; edges colored by shared Mass2Motifs via the 'interaction' column with Discrete Mapping) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- The exported .graphml loads in Cytoscape without errors and edge 'interaction' attributes and node 'TopSharedMotifs' are populated
- The merged 'matches' object contains both 'Scan' and 'SMILES' columns and successfully combines gnpslib, nap (and optionally derep) sources
- SMILES-to-InChIKey conversion via molconvert yields InChIKeys accepted by ClassyFire, producing a non-empty chemical-class dataframe (final)
- Molecular families that previously had only single GNPS library matches acquire motif annotations (e.g. Euphorbia-backbone and nicotinoyl-sidechain Mass2Motifs across two diterpenoid families totaling 73 features at MSI annotation level 3)
- Edges and nodes TSVs import correctly into Cytoscape with CLUSTERID1 as Source, 'interact' as Interaction Type, and CLUSTERID2 as Target

## Limitations

- The MS2LDA motif-document summary download can trigger a server connection timeout for large files; the file must then be exported manually from ms2lda.org
- The ms2lda.org summary table contains filtered motif-document relations using the thresholds set in the web app's Experimental Options tab, so probability/overlap thresholds should be set consistently there before harmonization
- Integration across tools is inherently fragile because each tool has its own output format and efficient cross-tool data matching is lacking, which is why explicit merging and SMILES→InChIKey conversion steps are required

## Evidence

- [abstract] integration is hampered as each tool has its own output format and efficient matching of data across these tools is lacking: "integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking"
- [methods] molconvert SMILES to InChIKeys: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
- [methods] matches object merging: "The object 'gnpslib' contains all GNPS library hits:

 `matches = [gnpslib, nap, derep]`"
- [readme] MS2LDA timeout manual download: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
- [readme] motif mapping parameters: "prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
