---
name: chemical-graph-parsing-and-featurization
description: Use when a task requires converting molecular structures (SMILES/MolBlock/SDF
  from spectral libraries like NIST20) into molecular graphs and heavy-atom-only skeletons
  H(G) as input to bond-breaking fragmentation or GNN featurization.
when_to_use_negative:
- When the input is already a feature table or fingerprint rather than a molecular
  structure — parsing/featurization is redundant
- When explicit hydrogen positions and isotope-exchange chemistry must be modeled
  directly, since stripping to H(G) defers hydrogen accounting to the hydrogen-tolerance
  enumeration (j = 4) rather than representing hydrogens explicitly
- When the task is spectral peak matching or retrieval ranking — those consume the
  featurized graph as input but are distinct skills
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Parses precursor molecular structures into molecular graphs and performs hydrogen
    stripping and molecule featurization during preprocessing
- name: Python
  role: Implementation language for the preprocessing and featurization scripts
- name: Pytorch Geometric
  role: Consumes the featurized molecular graph in the Molecule GNN stage (g_theta^MOL)
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:431ad0cb75f30db2034465adb262c047f82d54808af5e5a869cfc0a24fcda371
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - Python
  - Pytorch Geometric
  - RDKit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chemical-graph-parsing-and-featurization

## Summary

Parses precursor molecular structures into molecular graphs with RDKit and strips hydrogen atoms to produce the heavy atom skeleton H(G), yielding atom/bond features that downstream fragmentation DAG and GNN stages consume.

## When to use

Use when a task requires converting molecular structures (SMILES/MolBlock/SDF from spectral libraries like NIST20) into molecular graphs and heavy-atom-only skeletons H(G) as input to bond-breaking fragmentation or GNN featurization.

## When NOT to use

- When the input is already a feature table or fingerprint rather than a molecular structure — parsing/featurization is redundant
- When explicit hydrogen positions and isotope-exchange chemistry must be modeled directly, since stripping to H(G) defers hydrogen accounting to the hydrogen-tolerance enumeration (j = 4) rather than representing hydrogens explicitly
- When the task is spectral peak matching or retrieval ranking — those consume the featurized graph as input but are distinct skills

## Inputs

- Molecular structure records (SMILES or MolBlock/SDF) for NIST20 precursor molecules
- Exported NIST20 MS/MS spectral library metadata linking spectra to precursor structures
- RDKit molecular graph objects (atoms, bonds, atom/bond features)

## Outputs

- Heavy atom skeleton H(G): hydrogen-stripped, connected molecular graph (Definition 4.2)
- Atom- and bond-level feature tensors suitable for Pytorch Geometric GNN input
- Subgraph identity hashes from approximate Weisfeiler-Lehman hashing

## How to apply

Parse each precursor molecule into a molecular graph G using RDKit (the paper's preprocessing and featurization tool), reading the exported NIST20 structure records. Strip hydrogen atoms to obtain the heavy atom skeleton H(G), defined as the largest connected subgraph of G whose atoms are all non-hydrogen (Definition 4.2); the fragmentation DAG construction operates on H(G), so hydrogen handling is deferred to the hydrogen-tolerance enumeration (j = 4) applied later at fragment nodes. Featurize atoms and bonds with RDKit descriptors so the Molecule GNN (g_theta^MOL) can operate on the input graph, and apply the approximate Weisfeiler-Lehman hashing algorithm to subgraphs to assign consistent fragment identities. Validate that the parsed graph preserves the molecule's atom counts and connectivity before passing H(G) to the recursive bond-breaking algorithm.

## Related tools

- **RDKit** (Parses precursor molecular structures into molecular graphs and performs hydrogen stripping and molecule featurization during preprocessing)
- **Python** (Implementation language for the preprocessing and featurization scripts)
- **Pytorch Geometric** (Consumes the featurized molecular graph in the Molecule GNN stage (g_theta^MOL))

## Examples

```
from rdkit import Chem; mol = Chem.RemoveHs(Chem.MolFromMolBlock(mol_block)); heavy_atoms = mol.GetNumAtoms()
```

## Evaluation signals

- H(G) contains no hydrogen atoms: every node's element attribute differs from H
- H(G) is connected and is the largest connected heavy-atom subgraph of the parsed G
- Atom counts per element in the parsed graph match the molecule's known formula (NIST20 expert-curated formula annotations can serve as a cross-check)
- Featurization produces consistent feature dimensions across all molecules so Pytorch Geometric batching does not fail
- WL-hashed subgraph identities are consistent across molecules sharing identical fragments

## Limitations

- Stripping hydrogens means hydrogen counts must be recovered indirectly via the hydrogen tolerance enumeration (h_n - j to h_n + j with j = 4) at fragment nodes, so hydrogen placement is approximate
- Complex chemistry such as cyclizations can dramatically expand the fragment search space and is challenging to model, so featurized graphs alone do not capture all fragment chemistry
- The pipeline was validated on merged NIST20 spectra under Orbitrap HCD, [M+H]+ adduct conditions; broader experimental conditions were not evaluated

## Evidence

- [methods] RDKit parses and featurizes molecules before graph construction: [evidence span withheld — anchor status: not_found]
- [methods] Heavy atom skeleton H(G) definition (Definition 4.2) used for hydrogen stripping: [evidence span withheld — anchor status: not_found]
- [supplementary] Subgraph identity assignment via approximate WL hashing: "by applying an approximate Weisfeiler-Lehman hashing algorithm"
- [readme] Parsed and featurized graphs feed the GNN-based spectrum prediction pipeline: [evidence span withheld — anchor status: not_found]
