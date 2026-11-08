---
name: bridge-and-cut-detection-in-molecular-graphs
description: Use when generating in silico MS/MS fragmentation candidates for small
  molecules from structure databases (e.g., SMILES/SDF, 0-2000 Da), especially when
  enumerating all possible single- and double-bond disconnections efficiently is needed
  for spectral matching.
when_to_use_negative:
- The input is already a peak list or mass spectrum rather than a molecular structure
  — this skill generates fragments from structures, not from spectra
- 'Accurate prediction requires modeling resonance-aware fragmentation: molDiscovery
  naively treats benzene rings and large resonant structures as alternating single/double
  bonds'
- The molecule requires fragmentation rules beyond N-C, O-C, and C-C bond disconnections,
  since only a limited number of bond types and rearrangement rules are covered
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: molDiscovery
  role: 'Implements the fragmentation graph construction: builds metabolite graphs,
    runs bridge/2-cut detection (Hopcroft-Tarjan), and produces fragments for spectral
    matching in its database search pipeline'
  repo: github.com/mohimanilab/molDiscovery
- name: RDKit
  role: Parses SMILES/SDF molecular structures from the structure database into molecular
    graphs before cut detection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:2aded3f073381381e4198c9174e0c28edd39ce99621dcaa50f014ac76b731ce2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bridge-and-cut-detection-in-molecular-graphs

## Summary

An efficient graph algorithm used by molDiscovery to generate mass spectrometry fragmentations: it detects bridges and 2-cuts in molecular graphs (atoms as nodes, bonds as edges) to enumerate single- and double-bond-cut fragments for spectral matching in database search of small molecules.

## When to use

Use when generating in silico MS/MS fragmentation candidates for small molecules from structure databases (e.g., SMILES/SDF, 0-2000 Da), especially when enumerating all possible single- and double-bond disconnections efficiently is needed for spectral matching.

## When NOT to use

- The input is already a peak list or mass spectrum rather than a molecular structure — this skill generates fragments from structures, not from spectra
- Accurate prediction requires modeling resonance-aware fragmentation: molDiscovery naively treats benzene rings and large resonant structures as alternating single/double bonds
- The molecule requires fragmentation rules beyond N-C, O-C, and C-C bond disconnections, since only a limited number of bond types and rearrangement rules are covered

## Inputs

- Molecular structures in SMILES or SDF format from structure databases (e.g., PubChem, Dictionary of Natural Products)
- Molecular graphs with atoms as nodes and bonds as edges (molecules up to 2000 Da)

## Outputs

- Fragments from single-bond cuts (bridges), encoded as binary indicator vectors over the molecule's atoms
- Depth-n fragments from double-bond cuts (2-cuts), computed as bitwise intersections of parent fragment vectors
- Candidate fragment masses used for spectral matching against tandem mass spectra

## How to apply

Represent each molecule as a molecular graph with atoms as nodes and bonds as edges, loaded from SMILES/SDF via RDKit/molDiscovery parsers for molecules up to 2000 Da. Disconnect N-C, O-C, and C-C bonds to define candidate fragmentation edges, then run Hopcroft-Tarjan bridge and 2-cut detection to identify single and double bond cuts that yield connected fragments. Encode each single-bond-cut fragment as a binary indicator vector over the molecule's atoms, and build depth-n fragments by computing the bitwise intersection of parent fragment vectors for all combinations of n cut edges. The rationale is that bridges and 2-cuts capture exactly the single and double disconnections relevant to MS/MS fragmentation while avoiding exhaustive bond removal, making fragmentation generation efficient enough to search millions of structures. Finally, feed the resulting fragments into molDiscovery's probabilistic model for matching against tandem mass spectra such as GNPS CID/HCD data.

## Related tools

- **molDiscovery** (Implements the fragmentation graph construction: builds metabolite graphs, runs bridge/2-cut detection (Hopcroft-Tarjan), and produces fragments for spectral matching in its database search pipeline) — github.com/mohimanilab/molDiscovery
- **RDKit** (Parses SMILES/SDF molecular structures from the structure database into molecular graphs before cut detection)

## Evaluation signals

- Every depth-n fragment equals the bitwise intersection of valid parent (single-bond-cut) fragment vectors
- The pipeline completes without error for test molecules spanning the 0-2000 Da range
- All detected cuts correspond to disconnected N-C, O-C, or C-C bond edges in the molecular graph
- Fragments generated from a known molecule yield plausible fragment masses when matched against its reference tandem mass spectrum
- Downstream database search produces identifications at the reported 0% FDR control level

## Limitations

- molDiscovery naively treats benzene rings or large resonant structures as alternate single bonds and double bonds
- Only a limited number of bond types and rearrangement rules are covered; local/global structural features such as moieties are ignored
- Lower accuracy in low mass ranges (<600 Da) because fewer fragments are generated for small molecules
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra

## Evidence

- [intro] we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations and (ii) developing a probabilistic model: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations and (ii) developing a probabilistic model"
- [abstract] Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [discussion] molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds: "molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds"
- [readme] To test molDiscovery for academic users, please visit :: "To test molDiscovery for academic users, please visit :"
