---
name: molecular-network-node-annotation-mapping
description: Use when a GNPS molecular network has node-level structural annotations
  (SMILES from GNPS library hits, NAP, or DEREPLICATOR) and you need per-family chemical
  class consensus scores across the ClassyFire ontology.
when_to_use_negative:
- You want to map MS2LDA Mass2Motif substructural information to the network — that
  is a separate pyMolNetEnhancer workflow with its own prob/overlap/top parameters
- Node annotations lack SMILES strings, so no InChIKeys can be generated for ClassyFire
  classification
- The molecular network was not created through GNPS and node scan/component identifiers
  cannot be matched to annotations
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3332
tools:
- name: pyMolNetEnhancer
  role: Python module that loads annotation tables and computes per-family, per-ontology-level
    chemical class scores within the molecular network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: ClassyFire
  role: Automated chemical classification of node InChIKeys into superclass, class,
    and subclass categories
- name: molconvert
  role: ChemAxon command-line tool used to convert annotated node SMILES to InChIKeys
    prior to ClassyFire submission
- name: GNPS
  role: Platform whose molecular networks and library-hit annotations (with Scan and
    SMILES columns) supply the nodes and candidate structures being classified
- name: Network Annotation Propagation
  role: In silico structure annotation tool whose SMILES outputs feed the matches
    list for classification
- name: DEREPLICATOR
  role: In silico structure annotation tool whose SMILES outputs feed the matches
    list for classification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:da869cd070fe9c93156cced92201a91402247701a3495501d2ecf3d5262699da
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
  - molconvert
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# molecular-network-node-annotation-mapping

## Summary

Maps chemical class annotations onto GNPS mass spectral molecular network nodes by converting annotated SMILES to InChIKeys, classifying them with ClassyFire, and computing per-molecular-family chemical class scores at each ontology level (superclass, class, subclass).

## When to use

Use when a GNPS molecular network has node-level structural annotations (SMILES from GNPS library hits, NAP, or DEREPLICATOR) and you need per-family chemical class consensus scores across the ClassyFire ontology.

## When NOT to use

- You want to map MS2LDA Mass2Motif substructural information to the network — that is a separate pyMolNetEnhancer workflow with its own prob/overlap/top parameters
- Node annotations lack SMILES strings, so no InChIKeys can be generated for ClassyFire classification
- The molecular network was not created through GNPS and node scan/component identifiers cannot be matched to annotations

## Inputs

- Tab-separated structural annotation tables with 'Scan' and 'SMILES' columns from GNPS library hits
- Network Annotation Propagation (NAP) output annotations
- DEREPLICATOR output annotations
- Molecular network with network component indices (molecular family assignments)

## Outputs

- ClassyFireScores.tsv — per-molecular-family, per-ontology-level chemical class labels and classification scores in [0,1]
- InChIKeys derived from annotated node SMILES (intermediate for ClassyFire submission)

## How to apply

Load the candidate structure annotations (GNPS library hits plus NAP and DEREPLICATOR outputs) as tab-separated dataframes into a 'matches' list, where each entry has 'Scan' and 'SMILES' columns as required by pyMolNetEnhancer. Convert all SMILES to InChIKeys using ChemAxon's molconvert, adding the Marvin bin directory to the PATH environment variable. Submit the InChIKeys to ClassyFire for automated chemical classification at each ontology level (superclass, class, subclass). For each molecular family (network component index) and each ontology level, count the nodes attributed to each ClassyFire category and compute the score as the fraction of nodes belonging to the most abundant class — in the paper's Figure 2 worked example, the family is flavonoids with a class-level score of 0.375 (2.25/6). Write the per-family, per-level scores to a tab-separated output file ClassyFireScores.tsv.

## Related tools

- **pyMolNetEnhancer** (Python module that loads annotation tables and computes per-family, per-ontology-level chemical class scores within the molecular network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **ClassyFire** (Automated chemical classification of node InChIKeys into superclass, class, and subclass categories)
- **molconvert** (ChemAxon command-line tool used to convert annotated node SMILES to InChIKeys prior to ClassyFire submission)
- **GNPS** (Platform whose molecular networks and library-hit annotations (with Scan and SMILES columns) supply the nodes and candidate structures being classified)
- **Network Annotation Propagation** (In silico structure annotation tool whose SMILES outputs feed the matches list for classification)
- **DEREPLICATOR** (In silico structure annotation tool whose SMILES outputs feed the matches list for classification)

## Examples

```
matches = [gnpslib, nap, derep]
```

## Evaluation signals

- ClassyFireScores.tsv exists and is tab-separated
- All chemical classification scores fall within the range [0,1]
- The Figure 2 worked example family is classified as flavonoids with a class-level score of exactly 0.375 (2.25/6)
- Every molecular family (network component index) has score entries at each ClassyFire ontology level (superclass, class, subclass)
- SMILES-to-InChIKey conversion succeeded for all annotated scans (molconvert with Marvin bin on PATH)

## Limitations

- ClassyFire is an external web service, so server connection timeouts may occur (an analogous timeout issue is documented for MS2LDA job files in the README)
- Differing output formats across in silico annotation tools require harmonization into the pyMolNetEnhancer 'Scan'/'SMILES' schema before classification
- The score only reflects the most abundant class among annotated nodes; chemical class correctness of the underlying annotations requires expert review

## Evidence

- [methods] molconvert SMILES to InChIKeys conversion: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
- [abstract] workflow combining annotation tools with ClassyFire: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR)"
- [readme] pyMolNetEnhancer purpose statement: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [methods] matches list of annotation dataframes: "matches = [gnpslib, nap, derep]"
