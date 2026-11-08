---
name: molecular-network-chemical-class-annotation
description: Use when you have a GNPS molecular networking output (classical or feature-based)
  plus SMILES annotations (library hits, NAP, DEREPLICATOR) and want ClassyFire chemical
  class labels mapped onto the network graph.
when_to_use_negative:
- The dataset has no SMILES annotations (no GNPS library hits, NAP or DEREPLICATOR
  results) — ClassyFire cannot be queried without structures
- You only want structural motif substructure info without chemical taxonomy — the
  standalone MS2LDA-to-network mapping workflow suffices
- Input is not a GNPS molecular network (e.g. an already-aggregated feature table
  with no graph or scan IDs to join on)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Python module that maps ClassyFire chemical class and MS2LDA Mass2Motif information
    onto GNPS molecular networks and exports annotated .graphml/.tsv outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same chemical class / motif mapping workflow in
    R
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: ClassyFire
  role: Automated chemical classification source; provides superclass/class/subclass
    taxonomy and scores for InChIKeys derived from SMILES
- name: ChemAxon molconvert
  role: Converts SMILES from annotation tables to InChIKeys for ClassyFire submission
- name: MS2LDA
  role: Supplies Mass2Motif substructure annotations that can be co-mapped with chemical
    classes onto the network
- name: Network Annotation Propagation
  role: In silico structure annotation source whose SMILES outputs feed the ClassyFire
    classification
- name: DEREPLICATOR
  role: Alternative in silico annotation source whose SMILES outputs feed the ClassyFire
    classification
- name: GNPS
  role: Platform generating the molecular network (.graphml) and library hits whose
    node identifiers are joined on the 'Scan' column
- name: Cytoscape
  role: Visualization of the annotated network; color nodes by CF_subclass fill or
    TopSharedMotifs, edges by discrete mapping on 'interaction'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:0c6992c0e38b795ddf9d38b256c2b59639b914fe69963dd8f219d542d63afed3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ChemAxon molconvert
  - ClassyFire
  - Cytoscape
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# molecular-network-chemical-class-annotation

## Summary

Annotate GNPS molecular network nodes with hierarchical chemical taxonomy (ClassyFire superclass/class/subclass) by combining in silico annotation outputs (NAP, DEREPLICATOR, GNPS library hits) and optionally MS2LDA Mass2Motifs, using pyMolNetEnhancer or RMolNetEnhancer, and export annotated .graphml/.tsv files for Cytoscape visualization.

## When to use

Use when you have a GNPS molecular networking output (classical or feature-based) plus SMILES annotations (library hits, NAP, DEREPLICATOR) and want ClassyFire chemical class labels mapped onto the network graph.

## When NOT to use

- The dataset has no SMILES annotations (no GNPS library hits, NAP or DEREPLICATOR results) — ClassyFire cannot be queried without structures
- You only want structural motif substructure info without chemical taxonomy — the standalone MS2LDA-to-network mapping workflow suffices
- Input is not a GNPS molecular network (e.g. an already-aggregated feature table with no graph or scan IDs to join on)

## Inputs

- Tab-separated GNPS library hits table with 'Scan' and 'SMILES' columns
- Tab-separated NAP or DEREPLICATOR in silico annotation output with 'Scan' and 'SMILES' columns (or another in silico tool of preference)
- GNPS molecular networking job output (.graphml network, classical or feature-based)
- Optional: ms2lda.org Mass2Motif summary table for the LDA experiment built from the MGF clustered spectra (classical) or the MZmine-generated MGF (feature-based)
- ChemAxon MarvinSuite installation (molconvert) and ClassyFire access

## Outputs

- Annotated molecular network .graphml with CF_superclass, CF_class, CF_subclass node attributes (and *_score columns)
- Chemical-class mapping table (.txt) per node
- Combined chemical class + Mass2Motif GraphML (make_classyfire_graphml with motif-mapped network 'MG' and class dataframe 'final')
- Optional Cytoscape-ready 'Mass2Motifs_Nodes_*.tsv' and 'Mass2Motifs_Edges_*.tsv' files

## How to apply

Assemble a 'matches' dataframe list where each tab-separated input contains a numeric 'Scan' column matching GNPS node identifiers and a 'SMILES' column — typically the GNPS library hits table plus NAP and DEREPLICATOR in silico annotation results. Convert each SMILES to InChIKeys with ChemAxon's molconvert (adding the MarvinSuite bin path to PATH), retrieve ClassyFire superclass/class/subclass taxonomy with scores for each InChIKey, then run pyMolNetEnhancer to map the class columns onto the network GraphML, prefixing class columns with 'CF_' (CF_superclass, CF_class, CF_subclass plus corresponding *_score columns). To additionally propagate shared MS2LDA Mass2Motifs, load the ms2lda.org motif summary and map motifs with prob >= 0.01 and overlap >= 0.3 (thresholds adjustable in the ms2lda web app's Experimental Options tab; the summary table already reflects those filters), showing the top 5 most shared motifs per network component. Export the annotated .graphml and combined .txt table; import into Cytoscape and color nodes by 'CF_subclass' or by 'TopSharedMotifs' via Image/Chart, coloring edges by discrete mapping on 'interaction'.

## Related tools

- **pyMolNetEnhancer** (Python module that maps ClassyFire chemical class and MS2LDA Mass2Motif information onto GNPS molecular networks and exports annotated .graphml/.tsv outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same chemical class / motif mapping workflow in R) — https://github.com/madeleineernst/RMolNetEnhancer
- **ClassyFire** (Automated chemical classification source; provides superclass/class/subclass taxonomy and scores for InChIKeys derived from SMILES)
- **ChemAxon molconvert** (Converts SMILES from annotation tables to InChIKeys for ClassyFire submission)
- **MS2LDA** (Supplies Mass2Motif substructure annotations that can be co-mapped with chemical classes onto the network)
- **Network Annotation Propagation** (In silico structure annotation source whose SMILES outputs feed the ClassyFire classification)
- **DEREPLICATOR** (Alternative in silico annotation source whose SMILES outputs feed the ClassyFire classification)
- **GNPS** (Platform generating the molecular network (.graphml) and library hits whose node identifiers are joined on the 'Scan' column)
- **Cytoscape** (Visualization of the annotated network; color nodes by CF_subclass fill or TopSharedMotifs, edges by discrete mapping on 'interaction')

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- Every input table loaded into 'matches' has a numeric 'Scan' column that matches the GNPS node identifiers and a 'SMILES' column
- Output GraphML contains CF_superclass, CF_class, CF_subclass columns (with matching *_score columns), with the CF_ prefix
- The annotated .graphml loads into Cytoscape and CF_subclass is available as a node fill-color column
- Shared motifs between connected nodes are consistent with the Mass2Motif summary when prob >= 0.01 and overlap >= 0.3 filters are used
- For feature-based networks, the PyMolNetEnhancer and DEREPLICATOR outputs should be complementary — discrepancies are expected and both should be used

## Limitations

- ms2lda.org server connection timeouts may occur when fetching large summary files; the summary must be downloaded manually from the web app
- The MS2LDA summary table contains filtered motif–document relations using thresholds set in the web app's Experimental Options tab, so web-app thresholds must match the mapping parameters
- Mapping of Mass2Motifs onto networks requires the MarvinSuite bin path in PATH for molconvert
- DEREPLICATOR and ClassyFire/PyMolNetEnhancer annotations are complementary rather than redundant; relying on only one tool misses structures

## Evidence

- [methods] The object 'gnpslib' contains all GNPS library hits: "The object 'gnpslib' contains all GNPS library hits"
- [intro] pyMolNetEnhancer description: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [methods] molconvert SMILES-to-InChIKey: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
- [readme] CF_ prefix: [evidence span withheld — anchor status: not_found]
- [readme] prob/overlap defaults: "tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [discussion] DEREPLICATOR complementary: [evidence span withheld — anchor status: not_found]
