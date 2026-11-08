---
name: molecular-graph-fragment-generation
description: Use when you need to generate candidate fragment ions (m/z values) from
  a molecular structure (SMILES/SDF) for in silico spectral matching against tandem
  mass spectra, e.g. molecules from PubChem or DNP up to 2000 Da.
when_to_use_negative:
- You need to explain peaks caused by rearrangement rules or moieties — molDiscovery
  ignores local/global structural features and covers only a limited number of bond
  types and rearrangement rules
- The structure contains benzene rings or large resonant systems where correct bond
  order matters — molDiscovery naively treats them as alternating single/double bonds
- You require quantum-chemical or learned fragmentation spectra prediction rather
  than efficient graph-cut enumeration
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: molDiscovery
  role: 'Implements the fragmentation graph construction: metabolite graph building,
    bridge/2-cut detection, and depth-n fragment enumeration for database spectral
    search'
  repo: https://github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:d433994938a8d6ded3d51c823df7dcd9a97604229d62664829f6fca74e201a82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# molecular-graph-fragment-generation

## Summary

Efficiently enumerate mass-spectrometry fragments of a small molecule by representing it as a molecular graph, disconnecting only N-C, O-C, and C-C candidate bonds, and using a Hopcroft-Tarjan bridges-and-2-cuts algorithm to yield single- and double-bond-cut fragments as in molDiscovery.

## When to use

Use when you need to generate candidate fragment ions (m/z values) from a molecular structure (SMILES/SDF) for in silico spectral matching against tandem mass spectra, e.g. molecules from PubChem or DNP up to 2000 Da.

## When NOT to use

- You need to explain peaks caused by rearrangement rules or moieties — molDiscovery ignores local/global structural features and covers only a limited number of bond types and rearrangement rules
- The structure contains benzene rings or large resonant systems where correct bond order matters — molDiscovery naively treats them as alternating single/double bonds
- You require quantum-chemical or learned fragmentation spectra prediction rather than efficient graph-cut enumeration

## Inputs

- Molecular structures (SMILES or SDF) from a structure database such as PubChem or Dictionary of Natural Products (DNP), covering molecules up to 2000 Da
- Molecular graphs with atoms as nodes and bonds as edges

## Outputs

- Single-bond-cut fragments encoded as binary indicator vectors over the molecule's atoms
- Depth-n fragments computed as bitwise intersections of parent fragment vectors for combinations of n cut edges
- Candidate fragment m/z values used for downstream spectral matching

## How to apply

Load molecular structures (SMILES/SDF) from the structure database with RDKit/molDiscovery parsers and represent each molecule as a graph with atoms as nodes and bonds as edges. Disconnect all N-C, O-C, and C-C bonds to define candidate fragmentation edges, then run the Hopcroft-Tarjan algorithm to detect bridges and 2-cuts, which identify single- and double-bond cuts yielding fragments. Encode each single-bond-cut fragment as a binary indicator vector over the molecule's atoms, and build depth-n fragments by taking the bitwise intersection of parent fragment vectors over all combinations of n cut edges. This graph-based enumeration is what makes molDiscovery efficient enough to score spectra against millions of database structures rather than applying fragmentation mechanism rules per molecule.

## Related tools

- **molDiscovery** (Implements the fragmentation graph construction: metabolite graph building, bridge/2-cut detection, and depth-n fragment enumeration for database spectral search) — https://github.com/mohimanilab/molDiscovery

## Evaluation signals

- Every depth-n fragment equals the bitwise intersection of its valid parent fragment indicator vectors
- Detected cut edges correspond exactly to bridges (single cuts) and 2-cuts (double cuts) under the Hopcroft-Tarjan algorithm on the molecular graph
- Only N-C, O-C, and C-C bonds are treated as candidate fragmentation edges
- The pipeline completes for test molecules spanning the 0-2000 Da mass range
- Downstream search identifies unique molecules (e.g. 3185 unique small molecules at 0% FDR on 8 million GNPS spectra)

## Limitations

- Lower identification accuracy below ~600 Da, partly because fewer fragments are generated for small molecules in this range
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Only a limited number of bond types and rearrangement rules are covered; local/global structural features such as moieties are ignored

## Evidence

- [intro] we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations"
- [results] constructs fragmentation graphs via bridges and 2-cuts: [evidence span withheld — anchor status: not_found]
- [methods] disconnected candidate fragmentation edges: [evidence span withheld — anchor status: not_found]
- [methods] depth-n fragments via bitwise intersection: [evidence span withheld — anchor status: not_found]
- [discussion] fewer fragments for small molecules: "This is partially because fewer fragments are generated for molecules in this range."
