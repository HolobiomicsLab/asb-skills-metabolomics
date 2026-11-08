---
name: substructure-annotation-of-molecular-families
description: Use when molecular families in a GNPS molecular network have no or few
  spectral library matches but MS2LDA motif data, NAP/DEREPLICATOR in silico annotations,
  and ClassyFire classification are available, and you need level-3 class or substructure-level
  annotation.
when_to_use_negative:
- Input molecular family nodes already carry GNPS spectral library matches at MSI
  level 2 or higher, so motif-based propagation adds little over direct annotation.
- The analysis goal is feature-based (FBMN) rather than classical molecular networking
  — use Mass2Motifs_2_Network_FeatureBased instead, as the workflow and outputs differ.
- No MS2LDA experiment or in silico structure annotations (NAP, DEREPLICATOR, or another
  tool) exist for the dataset; MolNetEnhancer's integration depends on these external
  outputs.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: MolNetEnhancer
  role: Workflow combining molecular networking, MS2LDA, in silico annotation, and
    ClassyFire outputs into an enhanced molecular network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: pyMolNetEnhancer
  role: Python module that maps Mass2Motifs and chemical class information onto mass
    spectral molecular networks; run via example notebooks
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform creating the classical or feature-based molecular network whose job
    ID is consumed by pyMolNetEnhancer
- name: MS2LDA
  role: Produces the motif-document summary of Mass2Motifs substructures mapped onto
    the network
- name: Network Annotation Propagation
  role: In silico structure annotation whose SMILES output is merged into the 'matches'
    object alongside GNPS library hits
- name: ClassyFire
  role: Automated chemical classification of InChIKeys used to build the chemical-class
    dataframe
- name: molconvert
  role: ChemAxon tool converting SMILES to InChIKeys before ClassyFire classification
- name: Cytoscape
  role: Visualization of the .graphml output; color edges by 'interaction' (Discrete
    Mapping) and nodes by 'TopSharedMotifs'
  repo: https://cytoscape.org/
- name: RMolNetEnhancer
  role: Analogous R implementation of the same mapping workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: DEREPLICATOR
  role: Alternative in silico annotation tool; its output is complementary to pyMolNetEnhancer
    and both should be used
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:941b5fd2b862f97353faacab334571e0bfd9f85f8779c5bd50f5903c4f851b5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - Cytoscape
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# substructure-annotation-of-molecular-families

## Summary

Map MS2LDA Mass2Motif substructural annotations onto GNPS molecular networks with pyMolNetEnhancer, raising annotation of molecular families that lack GNPS spectral library hits to MSI level 3 (putatively characterized compound classes), as demonstrated for two Euphorbia diterpenoid molecular families covering 73 molecular features.

## When to use

Use when molecular families in a GNPS molecular network have no or few spectral library matches but MS2LDA motif data, NAP/DEREPLICATOR in silico annotations, and ClassyFire classification are available, and you need level-3 class or substructure-level annotation.

## When NOT to use

- Input molecular family nodes already carry GNPS spectral library matches at MSI level 2 or higher, so motif-based propagation adds little over direct annotation.
- The analysis goal is feature-based (FBMN) rather than classical molecular networking — use Mass2Motifs_2_Network_FeatureBased instead, as the workflow and outputs differ.
- No MS2LDA experiment or in silico structure annotations (NAP, DEREPLICATOR, or another tool) exist for the dataset; MolNetEnhancer's integration depends on these external outputs.

## Inputs

- MGF clustered spectra from MASSIVE (e.g. MSV000081082)
- GNPS molecular networking job output (GNPS job ID)
- MS2LDA experiment motif-document summary (MS2LDA job ID, e.g. experiment 390)
- NAP in silico structure annotation results (Scan/SMILES columns)
- GNPS library hits (Scan/SMILES columns)

## Outputs

- Enhanced molecular network GraphML with Mass2Motif and chemical-class attributes
- Mass2Motifs_Edges_Classical.tsv (CLUSTERID1, interact, CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv with TopSharedMotifs attributes
- ClassyFire chemical-class annotations at MSI level 3 for previously unannotated features

## How to apply

Retrieve the MGF clustered spectra from MASSIVE (e.g. MSV000081082) together with the GNPS molecular networking job output, the MS2LDA motif-document summary (download manually from ms2lda.org if a server connection timeout occurs), and NAP in silico structure annotation results. Install pyMolNetEnhancer with `pip install pyMolNetEnhancer` and run the Mass2Motifs_2_Network_Classical notebook, supplying your GNPS job ID and MS2LDA job ID with mapping parameters prob=0.01 (minimal probability score), overlap=0.3 (minimal overlap score), and top=5 (most shared motifs per molecular family). Merge the GNPS library hits and NAP output into a 'matches' object (tab-separated with 'Scan' and 'SMILES' columns), convert SMILES to InChIKeys with ChemAxon's molconvert, and obtain automated chemical classification through ClassyFire. Combine the motif-mapped network with the chemical-class dataframe using make_classyfire_graphml and write the enhanced network via nx.write_graphml, exporting 'Mass2Motifs_Edges_Classical.tsv' and 'Mass2Motifs_Nodes_Classical.tsv' as intermediate products.

## Related tools

- **MolNetEnhancer** (Workflow combining molecular networking, MS2LDA, in silico annotation, and ClassyFire outputs into an enhanced molecular network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **pyMolNetEnhancer** (Python module that maps Mass2Motifs and chemical class information onto mass spectral molecular networks; run via example notebooks) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform creating the classical or feature-based molecular network whose job ID is consumed by pyMolNetEnhancer)
- **MS2LDA** (Produces the motif-document summary of Mass2Motifs substructures mapped onto the network)
- **Network Annotation Propagation** (In silico structure annotation whose SMILES output is merged into the 'matches' object alongside GNPS library hits)
- **ClassyFire** (Automated chemical classification of InChIKeys used to build the chemical-class dataframe)
- **molconvert** (ChemAxon tool converting SMILES to InChIKeys before ClassyFire classification)
- **Cytoscape** (Visualization of the .graphml output; color edges by 'interaction' (Discrete Mapping) and nodes by 'TopSharedMotifs') — https://cytoscape.org/
- **RMolNetEnhancer** (Analogous R implementation of the same mapping workflow) — https://github.com/madeleineernst/RMolNetEnhancer
- **DEREPLICATOR** (Alternative in silico annotation tool; its output is complementary to pyMolNetEnhancer and both should be used)

## Examples

```
pip install pyMolNetEnhancer  # then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb with GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5
```

## Evaluation signals

- The output .graphml file loads in Cytoscape and the two target molecular families (e.g. 73 Euphorbia diterpenoid features) carry the expected motif annotations (Euphorbia-backbone, nicotinoyl-sidechain) in TopSharedMotifs and 'interaction' edge attributes.
- The edges TSV imports into Cytoscape with 'CLUSTERID1' as Source Node, 'interact' as Interaction Type, and 'CLUSTERID2' as Target Node without schema errors.
- Features previously lacking GNPS library matches reach MSI annotation level 3 (putatively characterized compound classes) after chemical-class propagation.
- All Mass2Motif mappings meet the user-set thresholds: probability ≥ 0.01 and overlap ≥ 0.3; summary-table relations were already filtered by thresholds set in the ms2lda.org Experimental Options tab.
- Every merged SMILES in the 'matches' object converts successfully to an InChIKey via molconvert before ClassyFire queries.

## Limitations

- A server connection timeout may occur when fetching the MS2LDA summary for large jobs; download the file manually from ms2lda.org.
- The ms2lda.org summary table contains filtered motif-document relations using the thresholds set in the web app, so thresholds must be consistent between the web app and the notebook.
- Integration across tools is hampered because each tool has its own output format and efficient matching of data across tools is lacking — hence the Scan/SMILES merge step.
- pyMolNetEnhancer and DEREPLICATOR outputs are complementary, so relying on only one tool under-annotates molecular networks.

## Evidence

- [abstract] MolNetEnhancer combines molecular networking, MS2LDA, and ClassyFire: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical"
- [readme] Mass2Motif mapping parameters: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br> <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] MS2LDA manual download fallback: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
- [intro] pyMolNetEnhancer purpose: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular"
- [readme] Cytoscape visualization of TopSharedMotifs: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left and select Mass2Motifs shown in 'TopSharedMotifs' in"
- [methods] SMILES to InChIKey conversion: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
