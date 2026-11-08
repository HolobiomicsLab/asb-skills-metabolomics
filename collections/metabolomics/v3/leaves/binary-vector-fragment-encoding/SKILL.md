---
name: binary-vector-fragment-encoding
description: Use when generating candidate fragmentations for a small molecule (SMILES/SDF,
  0–2000 Da) to support spectral matching or in silico database search against tandem
  MS spectra.
when_to_use_negative:
- The goal is predicting complete fragmentation spectra rather than generating fragments
  for database search scoring — molDiscovery generates fragment graphs, not predicted
  spectra like QCEIMS or CFM-ID
- Molecules whose dominant fragmentation requires bond types or rearrangements outside
  the N-C/O-C/C-C cut set, since molDiscovery 'only covers a limited number of bond
  types and rearrangement rules'
- Very small molecules (<400 Da), where fewer fragments are generated and identification
  accuracy drops
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: 'Implements the bridges-and-2-cuts fragmentation graph construction: builds
    the metabolite graph, runs Hopcroft-Tarjan bridge/2-cut detection, and encodes
    fragments as binary vectors for depth-n fragment generation'
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:b0e7086d83d1f6e1859adbf6064b3d6714c81f92327fa80d091ca8961bcb7129
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binary-vector-fragment-encoding

## Summary

Encodes mass-spectrometry fragmentation fragments of small molecules as binary indicator vectors over a molecule's atoms, enabling efficient depth-n fragment generation via bitwise intersection after Hopcroft-Tarjan bridge/2-cut detection. This is the graph-construction core that makes molDiscovery's database search tractable for molecules up to ~2000 Da.

## When to use

Use when generating candidate fragmentations for a small molecule (SMILES/SDF, 0–2000 Da) to support spectral matching or in silico database search against tandem MS spectra.

## When NOT to use

- The goal is predicting complete fragmentation spectra rather than generating fragments for database search scoring — molDiscovery generates fragment graphs, not predicted spectra like QCEIMS or CFM-ID
- Molecules whose dominant fragmentation requires bond types or rearrangements outside the N-C/O-C/C-C cut set, since molDiscovery 'only covers a limited number of bond types and rearrangement rules'
- Very small molecules (<400 Da), where fewer fragments are generated and identification accuracy drops

## Inputs

- Molecular structures in SMILES or SDF format from a structure database (e.g., PubChem, Dictionary of Natural Products)
- Molecules in the 0–2000 Da range
- Molecular graph with atoms as nodes and bonds as edges (built with RDKit/molDiscovery parsers)

## Outputs

- Fragmentation graph of depth-n fragments
- Binary indicator vectors over the molecule's atoms, one per single-bond-cut fragment
- Depth-n fragments computed as bitwise intersections of parent fragment vectors for all combinations of n cut edges

## How to apply

Load molecular structures (SMILES/SDF) from the structure database and represent each molecule as a molecular graph with atoms as nodes and bonds as edges. Disconnect all N-C, O-C, and C-C bonds to define candidate fragmentation edges, then run the Hopcroft-Tarjan algorithm to detect bridges and 2-cuts, which identify the single- and double-bond cuts that yield fragments. Encode each single-bond-cut fragment as a binary indicator vector over the molecule's atoms. Build depth-n fragments by computing the bitwise intersection of parent fragment vectors for all combinations of n cut edges; using bitsets rather than explicit subgraph enumeration is what keeps generation efficient for heavy molecules (>1000 Da) where rule-based and ML alternatives become computationally insufficient.

## Related tools

- **molDiscovery** (Implements the bridges-and-2-cuts fragmentation graph construction: builds the metabolite graph, runs Hopcroft-Tarjan bridge/2-cut detection, and encodes fragments as binary vectors for depth-n fragment generation) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- Every depth-n fragment equals the bitwise intersection of valid parent fragment vectors for that combination of n cut edges
- The pipeline completes end-to-end for test molecules spanning the 0–2000 Da range without graph-generation failures
- Single-bond-cut fragments correspond exactly to connected components obtained after cutting detected bridges in the molecular graph
- 2-cut fragments correspond to components after removing detected 2-cuts (pairs of candidate fragmentation edges)
- All generated fragments correspond to molecules produced only by cutting N-C, O-C, or C-C bonds as defined by the candidate fragmentation edge set

## Limitations

- molDiscovery naively treats benzene rings and large resonant structures as alternating single/double bonds, which can misassign candidate fragmentation edges
- molDiscovery ignores local/global structural features (e.g., moiety context) and covers only a limited number of bond types and rearrangement rules
- Lower identification accuracy below ~600 Da because fewer fragments are generated for small molecules
- Fragmentation accuracy depends on experimental factors (instrument type, dissociation technique, ionization mode, collision energy) that the default model only partially covers (combined CID/HCD positive-mode training)

## Evidence

- [intro] we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations"
- [discussion] molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds: "molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds"
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
