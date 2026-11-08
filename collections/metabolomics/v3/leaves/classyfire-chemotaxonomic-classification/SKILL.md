---
name: classyfire-chemotaxonomic-classification
description: Use when MS/MS molecular network nodes carry SMILES (from GNPS library
  hits, NAP, or DEREPLICATOR) and you need hierarchical chemical class assignments
  to compare metabolite class distributions across taxa or network components.
when_to_use_negative:
- The molecular network nodes lack SMILES annotations from any in silico tool or library
  — ClassyFire requires structures (via InChIKeys) and cannot classify consensus spectra
  alone
- You only need motif/substructure mapping without chemical taxonomy — use the Mass2Motifs-to-network
  workflow instead of the chemical-class workflow
- The goal is de novo structure elucidation rather than class-level annotation and
  chemotaxonomic comparison
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: ClassyFire
  role: Automated chemical classification providing superclass, class, subclass and
    scores for each annotated node
- name: pyMolNetEnhancer
  role: Python module that converts SMILES to InChIKeys, retrieves ClassyFire taxonomy,
    and maps CF_-prefixed class columns onto the GNPS network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for integrating chemical class information within mass
    spectral molecular networks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: Network Annotation Propagation
  role: In silico structure annotation supplying SMILES for nodes prior to classification
- name: DEREPLICATOR
  role: In silico structure annotation supplying SMILES for nodes prior to classification;
    its output is complementary to pyMolNetEnhancer's class mapping
- name: GNPS
  role: Platform generating the mass spectral molecular network and library hits whose
    'Scan' identifiers anchor the annotations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:f7debe49c0b646cb89a11724bf1b283f2d1e6aa4aaeed6434b0ac534b2cdd7d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - DEREPLICATOR
  - GNPS
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ClassyFire chemotaxonomic classification

## Summary

Retrieves ClassyFire chemical taxonomy (superclass/class/subclass plus scores) for molecular network nodes via InChIKeys, maps it onto the GNPS network with pyMolNetEnhancer as CF_-prefixed columns, and enables clade- or component-level chemotaxonomic distribution analysis.

## When to use

Use when MS/MS molecular network nodes carry SMILES (from GNPS library hits, NAP, or DEREPLICATOR) and you need hierarchical chemical class assignments to compare metabolite class distributions across taxa or network components.

## When NOT to use

- The molecular network nodes lack SMILES annotations from any in silico tool or library — ClassyFire requires structures (via InChIKeys) and cannot classify consensus spectra alone
- You only need motif/substructure mapping without chemical taxonomy — use the Mass2Motifs-to-network workflow instead of the chemical-class workflow
- The goal is de novo structure elucidation rather than class-level annotation and chemotaxonomic comparison

## Inputs

- Tab-separated annotation tables with numeric 'Scan' and 'SMILES' columns (GNPS library hits, NAP results, DEREPLICATOR results)
- GNPS feature-based or classical molecular networking output (e.g. for MASSIVE accession MSV000081805)
- MS2LDA experiment motif summary (e.g. MS2LDA experiment 566), downloaded manually if ms2lda.org connection times out
- Motif-mapped network object 'MG' and chemical class dataframe 'final' for make_classyfire_graphml

## Outputs

- Annotated .graphml with CF_superclass, CF_class, CF_subclass and *_score node attributes (loadable into Cytoscape with CF_subclass as node fill-color column)
- Annotated .txt table of node chemical classes
- Combined GraphML of chemical classes plus Mass2Motif substructure information
- Normalized CF_subclass distribution table per clade/network component

## How to apply

Load the GNPS library hits together with NAP and DEREPLICATOR in silico annotations into the 'matches' dataframe list, ensuring each tab-separated input contains a numeric 'Scan' column matching GNPS node identifiers and a 'SMILES' column. Convert SMILES to InChIKeys using ChemAxon's molconvert (add the MarvinSuite bin path to PATH), then retrieve ClassyFire superclass, class, subclass and corresponding scores for each annotated node. Map the chemical classes onto the network with pyMolNetEnhancer, labeling all class columns with the 'CF_' prefix (CF_superclass, CF_class, CF_subclass and *_score columns), and export the annotated .graphml and .txt table outputs. For substructure-aware chemotaxonomy, combine the class dataframe ('final') with motif-mapped Mass2Motif information using make_classyfire_graphml on the 'MG' network, with motif mapping defaults prob = 0.01, overlap = 0.3, top = 5. Aggregate CF_subclass assignments per clade or network component index and normalize to build class-distribution tables.

## Related tools

- **ClassyFire** (Automated chemical classification providing superclass, class, subclass and scores for each annotated node)
- **pyMolNetEnhancer** (Python module that converts SMILES to InChIKeys, retrieves ClassyFire taxonomy, and maps CF_-prefixed class columns onto the GNPS network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for integrating chemical class information within mass spectral molecular networks) — https://github.com/madeleineernst/RMolNetEnhancer
- **Network Annotation Propagation** (In silico structure annotation supplying SMILES for nodes prior to classification)
- **DEREPLICATOR** (In silico structure annotation supplying SMILES for nodes prior to classification; its output is complementary to pyMolNetEnhancer's class mapping)
- **GNPS** (Platform generating the mass spectral molecular network and library hits whose 'Scan' identifiers anchor the annotations)

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- All class columns carry the CF_ prefix (CF_superclass, CF_class, CF_subclass and corresponding *_score columns) in the output GraphML and txt table
- Every node with a SMILES input has a resolved InChIKey that returned ClassyFire superclass/class/subclass entries
- The .graphml loads into Cytoscape with 'CF_subclass' available as a node fill-color column
- Aggregated, normalized CF_subclass distributions reproduce the expected pattern (e.g. Rhamnoid clades enriched in flavonoids, carbohydrates and anthraquinones; Ziziphoid clades enriched in triterpenoids)
- Known motif-class pairs appear in the combined output, e.g. kaempferol-related Mass2Motif 130 (m/z 284/255/227) and quercetin-related Mass2Motifs 86/149

## Limitations

- molconvert requires ChemAxon MarvinSuite with its bin path added to PATH — without it the SMILES-to-InChIKey conversion step fails
- Class coverage depends on upstream in silico tools: pyMolNetEnhancer and DEREPLICATOR outputs are complementary, so researchers should use both tools to annotate their molecular networks
- The ms2lda.org summary download can hit a server connection timeout depending on file size and may need manual download

## Evidence

- [methods] ClassyFire retrieval step: [evidence span withheld — anchor status: not_found]
- [intro] pyMolNetEnhancer scope: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [discussion] complementarity of in silico annotations: [evidence span withheld — anchor status: not_found]
- [readme] motif mapping thresholds: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br> <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
