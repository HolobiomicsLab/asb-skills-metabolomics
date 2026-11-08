---
name: in-silico-structure-annotation-integration
description: Use when a GNPS molecular network (classical or feature-based) contains
  nodes lacking chemical class labels and you have tab-separated annotation tables
  with Scan and SMILES columns from library hits, NAP, or DEREPLICATOR to propagate
  ClassyFire taxonomy onto the network.
when_to_use_negative:
- The molecular network has not yet been generated via GNPS — build the network first,
  then annotate
- Inputs are MS/MS spectra without any in silico or library structure annotations
  (no SMILES available) — this skill needs SMILES-bearing annotation tables
- You want substructure/Motif-level (MS2LDA) annotation rather than chemical class
  propagation — use the separate Mass2Motif mapping workflow instead
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: GNPS
  role: Platform that generates the classical or feature-based mass spectral molecular
    network whose nodes are the targets of chemical annotation
  repo: https://gnps.ucsd.edu/
- name: Network Annotation Propagation
  role: In silico structure annotation tool whose output (SMILES per node) is one
    of the candidate annotation sources merged into the 'matches' list
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/nap/
- name: DEREPLICATOR
  role: In silico structure annotation tool whose output supplies additional SMILES
    annotations for network nodes; complementary to NAP and to the ClassyFire-based
    classification
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/dereplicator/
- name: ClassyFire
  role: Automated chemical classification of InChIKeys at superclass, class, and subclass
    levels; its categories define the propagated taxonomy
- name: molconvert
  role: ChemAxon tool converting SMILES to InChIKeys, the input required by ClassyFire;
    requires adding the MarvinSuite bin directory to PATH
  repo: https://docs.chemaxon.com/display/docs/Molconvert
- name: pyMolNetEnhancer
  role: Python module that maps the chemical class information onto the network, applies
    the CF_ column prefix, and writes .graphml/.tsv outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for integrating chemical class information within GNPS
    mass spectral molecular networks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:d70ed863dd184284fe15fcddb808704b72a416dc1f882fbb945666ac2885cd48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - DEREPLICATOR
  - GNPS
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# in-silico-structure-annotation-integration

## Summary

Integrate in silico structure annotations (GNPS library hits, NAP, DEREPLICATOR) into a GNPS mass spectral molecular network, converting SMILES to InChIKeys and retrieving ClassyFire chemical classes to build a per-node and per-molecular-family chemical taxonomy. This skill grounds MolNetEnhancer's chemical-annotation layer for metabolomics networks.

## When to use

Use when a GNPS molecular network (classical or feature-based) contains nodes lacking chemical class labels and you have tab-separated annotation tables with Scan and SMILES columns from library hits, NAP, or DEREPLICATOR to propagate ClassyFire taxonomy onto the network.

## When NOT to use

- The molecular network has not yet been generated via GNPS — build the network first, then annotate
- Inputs are MS/MS spectra without any in silico or library structure annotations (no SMILES available) — this skill needs SMILES-bearing annotation tables
- You want substructure/Motif-level (MS2LDA) annotation rather than chemical class propagation — use the separate Mass2Motif mapping workflow instead

## Inputs

- Molecular network output files from GNPS (classical or feature-based, with node identifiers and componentindex)
- GNPS library hits as tab-separated table with 'Scan' and 'SMILES' columns
- NAP in silico annotation results (tab-separated, with 'Scan' and 'SMILES')
- DEREPLICATOR in silico annotation results (tab-separated, with 'Scan' and 'SMILES')

## Outputs

- ClassyFireScores.tsv: per-family, per-level chemical-class scores (tab-separated, values in [0,1])
- Annotated network as .graphml with CF_superclass, CF_class, CF_subclass and corresponding *_score node columns
- Annotated node/edge tables as .tsv files loadable into Cytoscape

## How to apply

Load the candidate structure annotations into a 'matches' list of dataframes, each containing a numeric 'Scan' column matching GNPS node identifiers and a 'SMILES' column. Convert all SMILES to InChIKeys using ChemAxon's molconvert, adding the MarvinSuite bin directory to the PATH environment variable first. Submit the InChIKeys to ClassyFire for chemical classification at each ontology level (superclass, class, subclass). For each molecular family (network component index) and each level, compute the chemical classification score as the fraction of nodes in the most abundant class — the Figure 2 worked example gives a flavonoid class-level score of 0.375 (2.25/6). Label all class columns with the 'CF_' prefix (CF_superclass, CF_class, CF_subclass plus *_score) and export the annotated .graphml plus .txt table outputs via pyMolNetEnhancer. The rationale: comparison of PyMolNetEnhancer and DEREPLICATOR outputs shows they are complementary, so both tools should be used to annotate networks.

## Related tools

- **GNPS** (Platform that generates the classical or feature-based mass spectral molecular network whose nodes are the targets of chemical annotation) — https://gnps.ucsd.edu/
- **Network Annotation Propagation** (In silico structure annotation tool whose output (SMILES per node) is one of the candidate annotation sources merged into the 'matches' list) — https://ccms-ucsd.github.io/GNPSDocumentation/nap/
- **DEREPLICATOR** (In silico structure annotation tool whose output supplies additional SMILES annotations for network nodes; complementary to NAP and to the ClassyFire-based classification) — https://ccms-ucsd.github.io/GNPSDocumentation/dereplicator/
- **ClassyFire** (Automated chemical classification of InChIKeys at superclass, class, and subclass levels; its categories define the propagated taxonomy)
- **molconvert** (ChemAxon tool converting SMILES to InChIKeys, the input required by ClassyFire; requires adding the MarvinSuite bin directory to PATH) — https://docs.chemaxon.com/display/docs/Molconvert
- **pyMolNetEnhancer** (Python module that maps the chemical class information onto the network, applies the CF_ column prefix, and writes .graphml/.tsv outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for integrating chemical class information within GNPS mass spectral molecular networks) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- ClassyFireScores.tsv exists, is tab-separated, and every chemical classification score lies in [0,1]
- The Figure 2 worked example family's flavonoid class-level score equals 0.375 (2.25/6)
- The annotated .graphml loads into Cytoscape with 'CF_subclass' available as a node fill-color column
- Every annotation dataframe in the 'matches' list has a numeric 'Scan' column matching GNPS node identifiers and a 'SMILES' column
- CF_ prefixed class columns (CF_superclass, CF_class, CF_subclass plus *_score) are present in both the .graphml and .tsv outputs

## Limitations

- Integration across tools is hampered by differing output formats; each tool has its own format and efficient matching of data across tools is lacking, which is why this workflow standardizes on 'Scan' + 'SMILES' columns
- A server connection timeout may occur on ms2lda.org when retrieving large summaries; download the summary file manually as documented
- Deriving chemical classes via ClassyFire requires SMILES-to-InChIKey conversion via molconvert; if the MarvinSuite bin path is not on PATH, conversion fails
- Chemical class correctness of annotations is judged by expert review, not by an automated accuracy metric
- PyMolNetEnhancer and DEREPLICATOR outputs are complementary, so relying on only one tool leaves annotations incomplete

## Evidence

- [intro] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular Networking (GNPS) platform.: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular"
- [methods] Map chemical class information to mass spectral molecular networks: "## Map chemical class information to mass spectral molecular networks <a name="ChemicalClasses_to_Network"></a>"
- [discussion] Comparison of the output from PyMolNetEnhancer and DEREPLICATOR clearly shows that they are complementary, therefore, researchers should use both tools to annotate their molecular networks.: [evidence span withheld — anchor status: not_found]
- [readme] Install pyMolNetEnhancer with pip install pyMolNetEnhancer: "Install pyMolNetEnhancer with:

 `pip install pyMolNetEnhancer`"
- [methods] we use ChemAxon's molconvert to convert SMILES to InChIKeys: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
