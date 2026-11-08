---
name: classyfire-ontology-aggregation
description: Use when a GNPS molecular network's nodes carry candidate SMILES annotations
  (GNPS library hits, NAP, DEREPLICATOR) and you must compute per-family chemical
  class scores across the ClassyFire ontology hierarchy.
when_to_use_negative:
- The goal is mapping MS2LDA substructural (Mass2Motif) information onto the network
  rather than computing per-family chemical class scores
- Nodes lack SMILES annotations (e.g. only spectral library match names without structures),
  so no InChIKeys can be generated for ClassyFire submission
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3172
tools:
- name: ClassyFire
  role: Automated chemical classification of submitted InChIKeys at each level of
    the chemical ontology (superclass, class, subclass)
- name: pyMolNetEnhancer
  role: Python module that aggregates ClassyFire classes per molecular family (network
    component index) and computes the chemical classification scores written to ClassyFireScores.tsv
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: molconvert
  role: ChemAxon command-line tool used to convert node SMILES structures into InChIKeys
    prior to ClassyFire submission
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:cf6815f7b3759d444ee2ff7c580ee4ce729d1a2a44818f3db18bf904039cf968
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - MolNetEnhancer
  - molconvert
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classyfire-ontology-aggregation

## Summary

Aggregates per-node ClassyFire chemical classifications into a chemical classification score for each molecular family (network component index) at every level of the ClassyFire ontology (superclass, class, subclass), as implemented in MolNetEnhancer/pyMolNetEnhancer.

## When to use

Use when a GNPS molecular network's nodes carry candidate SMILES annotations (GNPS library hits, NAP, DEREPLICATOR) and you must compute per-family chemical class scores across the ClassyFire ontology hierarchy.

## When NOT to use

- The goal is mapping MS2LDA substructural (Mass2Motif) information onto the network rather than computing per-family chemical class scores
- Nodes lack SMILES annotations (e.g. only spectral library match names without structures), so no InChIKeys can be generated for ClassyFire submission

## Inputs

- Tab-separated annotation tables (GNPS library hits, NAP output, DEREPLICATOR output) each containing 'Scan' and 'SMILES' columns
- GNPS molecular network with componentindex/network component index per node
- SMILES structure strings for annotated nodes

## Outputs

- ClassyFireScores.tsv: per-family (network component index), per-ontology-level chemical class scores in [0,1]
- ClassyFire chemical classifications (superclass, class, subclass) per InChIKey

## How to apply

Load the candidate structure annotations as tab-separated dataframes into a matches list (e.g. matches = [gnpslib, nap, derep]), ensuring each entry has 'Scan' and 'SMILES' columns as required by pyMolNetEnhancer. Convert every SMILES to an InChIKey with ChemAxon's molconvert, adding the Marvin bin directory to the PATH environment variable first. Submit the InChIKeys to ClassyFire for automated classification at each ontology level (superclass, class, subclass). Then, for each molecular family identified by its network component index and each ontology level, count the nodes assigned to each ClassyFire category and compute the score as the fraction of nodes belonging to the most abundant class — in the paper's Figure 2 worked example the family is classified as flavonoids with a class-level score of 0.375 (2.25/6). Write the per-family, per-level scores to ClassyFireScores.tsv.

## Related tools

- **ClassyFire** (Automated chemical classification of submitted InChIKeys at each level of the chemical ontology (superclass, class, subclass))
- **pyMolNetEnhancer** (Python module that aggregates ClassyFire classes per molecular family (network component index) and computes the chemical classification scores written to ClassyFireScores.tsv) — https://github.com/madeleineernst/pyMolNetEnhancer
- **molconvert** (ChemAxon command-line tool used to convert node SMILES structures into InChIKeys prior to ClassyFire submission)

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- ClassyFireScores.tsv exists and is tab-separated
- All chemical class score values fall in the range [0,1]
- The worked-example molecular family's flavonoid class-level score equals 0.375 (2.25/6 nodes)
- Every family has score entries at superclass, class, and subclass ontology levels
- Chemical class correctness of the annotations is judged by expert review

## Limitations

- Score computation requires molconvert (Marvin bin directory on PATH) to succeed; SMILES that fail conversion yield no InChIKey and thus no ClassyFire classification
- Only nodes with candidate structure annotations contribute to a family's score; unannotated nodes cannot be classified
- ClassyFire submission depends on an external web service, so server timeouts/availability can interrupt classification of large batches

## Evidence

- [abstract] in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical classification through ClassyFire: "in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical classification through ClassyFire"
- [methods] Convert all SMILES structures to InChIKeys using ChemAxon's molconvert, ensuring the Marvin bin directory is added to the PATH environment variable.: [evidence span withheld — anchor status: not_found]
- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
