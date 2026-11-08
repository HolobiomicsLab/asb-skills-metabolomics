---
name: chemical-structure-classification
description: Use when a GNPS molecular network's nodes have structure annotations
  (GNPS library hits, NAP or DEREPLICATOR outputs with SMILES) and you need per-molecular-family
  chemical class scores at each ClassyFire ontology level.
when_to_use_negative:
- The task is mapping MS2LDA Mass2Motif substructure information onto the network
  (prob/overlap/top parameters), not chemical class scoring — use the Mass2Motifs_2_Network
  workflow instead.
- The input data has no structure annotations (no SMILES/InChIKeys) — ClassyFire classification
  cannot proceed without structural representations.
- Input is raw MS/MS spectra rather than an annotated molecular network; create the
  molecular network through GNPS and perform in silico annotation (NAP, DEREPLICATOR,
  or another tool of preference) first.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: pyMolNetEnhancer
  role: Python module that loads the matches list, orchestrates ClassyFire classification,
    and writes per-family chemical class scores
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for mapping chemical class information to mass spectral
    molecular networks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: ClassyFire
  role: Automated chemical classification of InChIKeys at superclass, class, and subclass
    ontology levels
- name: molconvert
  role: ChemAxon command-line tool converting annotation SMILES to InChIKeys prior
    to ClassyFire submission
  repo: https://docs.chemaxon.com/display/docs/Molconvert
- name: GNPS
  role: Source of library-hit structure annotations and of the molecular network whose
    component indices define molecular families
  repo: https://gnps.ucsd.edu/
- name: Network Annotation Propagation
  role: In silico structure annotation tool whose output SMILES feed the classification
    step
- name: DEREPLICATOR
  role: In silico structure annotation tool whose output SMILES feed the classification
    step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:21c4803125a47f5e954a7c6ccfa4f077daede42a630ece1a740375930683b6bd
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
  - molconvert
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# chemical-structure-classification

## Summary

Assign ClassyFire chemical classes to molecular-network nodes and compute per-family chemical classification scores at each ClassyFire ontology level (superclass, class, subclass), as implemented in MolNetEnhancer/pyMolNetEnhancer.

## When to use

Use when a GNPS molecular network's nodes have structure annotations (GNPS library hits, NAP or DEREPLICATOR outputs with SMILES) and you need per-molecular-family chemical class scores at each ClassyFire ontology level.

## When NOT to use

- The task is mapping MS2LDA Mass2Motif substructure information onto the network (prob/overlap/top parameters), not chemical class scoring — use the Mass2Motifs_2_Network workflow instead.
- The input data has no structure annotations (no SMILES/InChIKeys) — ClassyFire classification cannot proceed without structural representations.
- Input is raw MS/MS spectra rather than an annotated molecular network; create the molecular network through GNPS and perform in silico annotation (NAP, DEREPLICATOR, or another tool of preference) first.

## Inputs

- GNPS library hits table (tab-separated, with 'Scan' and 'SMILES' columns)
- Network Annotation Propagation (NAP) annotation output (tab-separated, with 'Scan' and 'SMILES' columns)
- DEREPLICATOR annotation output (tab-separated, with 'Scan' and 'SMILES' columns)
- Molecular network with network component indices identifying molecular families

## Outputs

- ClassyFireScores.tsv — per-family, per-ontology-level chemical class assignments and classification scores (tab-separated)
- Per-node ClassyFire annotations (superclass, class, subclass) mapped onto network component indices

## How to apply

Load the candidate structure annotations (GNPS library hits plus NAP and DEREPLICATOR outputs) as tab-separated dataframes into a matches list, with each entry containing 'Scan' and 'SMILES' columns as required by pyMolNetEnhancer. Convert all SMILES to InChIKeys with ChemAxon's molconvert, ensuring the Marvin bin directory is added to the PATH environment variable. Submit the InChIKeys to ClassyFire for automated chemical classification at each ontology level (superclass, class, subclass). For each molecular family (network component index) and each level, count nodes attributed to each ClassyFire category and compute the score as the fraction of nodes belonging to the most abundant class — in the Figure 2 worked example the family is classified as flavonoids with a class-level score of 0.375 (2.25/6). Write the per-family, per-level scores to a tab-separated output file ClassyFireScores.tsv.

## Related tools

- **pyMolNetEnhancer** (Python module that loads the matches list, orchestrates ClassyFire classification, and writes per-family chemical class scores) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for mapping chemical class information to mass spectral molecular networks) — https://github.com/madeleineernst/RMolNetEnhancer
- **ClassyFire** (Automated chemical classification of InChIKeys at superclass, class, and subclass ontology levels)
- **molconvert** (ChemAxon command-line tool converting annotation SMILES to InChIKeys prior to ClassyFire submission) — https://docs.chemaxon.com/display/docs/Molconvert
- **GNPS** (Source of library-hit structure annotations and of the molecular network whose component indices define molecular families) — https://gnps.ucsd.edu/
- **Network Annotation Propagation** (In silico structure annotation tool whose output SMILES feed the classification step)
- **DEREPLICATOR** (In silico structure annotation tool whose output SMILES feed the classification step)

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- ClassyFireScores.tsv exists and is tab-separated.
- All chemical classification scores fall in the range [0, 1], since each is a fraction of nodes.
- The Figure 2 worked example family reproduces a class-level score of 0.375 (2.25/6) for flavonoids.
- Every input SMILES yielded an InChIKey via molconvert (Marvin bin directory on PATH); nodes without a convertible structure are handled explicitly.
- The assigned dominant class per family matches expert review of the underlying annotations.

## Limitations

- Integration across tools is hampered because each tool has its own output format and efficient matching of data across tools is lacking — annotation tables must be normalized to 'Scan' and 'SMILES' columns before use.
- DEREPLICATOR is not mandatory: the workflow allows 'another tool of preference' for in silico structure annotation, so annotation coverage and quality vary with the chosen tool.
- pyMolNetEnhancer and DEREPLICATOR outputs are complementary rather than redundant, so class scores alone should not be treated as exhaustive annotation.

## Evidence

- [abstract] MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical"
- [abstract] integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking: "integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking"
- [methods] we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
- [methods] The object 'gnpslib' contains all GNPS library hits:

 `matches = [gnpslib, nap, derep]`: "The object 'gnpslib' contains all GNPS library hits:

 `matches = [gnpslib, nap, derep]`"
- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
