---
name: mass-spectrometry-fragmentation-algorithm-implementation
description: Use when you must enumerate all single- and double-bond-cleavage fragments
  of small molecules (0–2000 Da) from a structure database so tandem mass spectra
  can be matched against millions of candidate structures.
when_to_use_negative:
- The goal is spectral library search against reference spectra rather than in silico
  fragmentation of database structures.
- The molecule requires modeling rearrangement rules or local/global structural features
  (moieties), which molDiscovery currently ignores and only covers for a limited number
  of bond types.
- Accurate benzene-ring or large resonant-structure fragmentation is required, since
  molDiscovery naively treats them as alternating single and double bonds.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: molDiscovery
  role: 'Implements the fragmentation graph construction: parses SMILES/SDF structures,
    runs the bridge/2-cut algorithm, and generates fragments used for spectral matching
    in the database search pipeline.'
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:bfd461cf6661a46dad46a2eb12452541b7d97b96351e72277288f8bdee925695
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectrometry-fragmentation-algorithm-implementation

## Summary

Implements molDiscovery's efficient fragmentation-graph construction: a molecular graph is built from SMILES/SDF structures, N-C/O-C/C-C bonds are marked as candidate cut edges, and a Hopcroft-Tarjan bridge/2-cut algorithm plus bitwise intersection of fragment indicator vectors generates depth-n fragments for database search scoring.

## When to use

Use when you must enumerate all single- and double-bond-cleavage fragments of small molecules (0–2000 Da) from a structure database so tandem mass spectra can be matched against millions of candidate structures.

## When NOT to use

- The goal is spectral library search against reference spectra rather than in silico fragmentation of database structures.
- The molecule requires modeling rearrangement rules or local/global structural features (moieties), which molDiscovery currently ignores and only covers for a limited number of bond types.
- Accurate benzene-ring or large resonant-structure fragmentation is required, since molDiscovery naively treats them as alternating single and double bonds.

## Inputs

- Molecular structures in SMILES or SDF format from a small molecule database (e.g. PubChem, Dictionary of Natural Products)
- Molecular graph with atoms as nodes and bonds as edges

## Outputs

- Fragmentation graph of single- and double-bond-cut fragments (bridges and 2-cuts)
- Binary indicator vectors over the molecule's atoms for each single-bond-cut fragment
- Depth-n fragments computed as bitwise intersections of parent fragment vectors

## How to apply

Load each molecule (SMILES/SDF from e.g. PubChem or DNP) with the RDKit/molDiscovery parsers and represent it as a molecular graph with atoms as nodes and bonds as edges. Disconnect N-C, O-C, and C-C bonds to define the candidate fragmentation edges, then run the Hopcroft-Tarjan algorithm to find bridges and 2-cuts, which identify single- and double-bond cuts yielding fragments. Encode each single-bond-cut fragment as a binary indicator vector over the molecule's atoms, and build depth-n fragments as the bitwise intersection of parent fragment vectors over all combinations of n cut edges. This graph-based enumeration is what lets molDiscovery efficiently generate fragmentations for molecules spanning 0–2000 Da, where prior ML approaches were too slow for heavy molecules (>1000 Da) and inaccurate for tiny ones (<400 Da).

## Related tools

- **molDiscovery** (Implements the fragmentation graph construction: parses SMILES/SDF structures, runs the bridge/2-cut algorithm, and generates fragments used for spectral matching in the database search pipeline.) — github.com/mohimanilab/molDiscovery

## Examples

```
python molDiscovery.py --database structures.sdf --spectra query.mzml
```

## Evaluation signals

- Every depth-n fragment equals the bitwise intersection of its valid parent fragment indicator vectors.
- The pipeline completes without error for test molecules spanning the 0–2000 Da mass range.
- The set of single-bond-cut fragments matches the bridges found by Hopcroft-Tarjan on the molecular graph with N-C/O-C/C-C edges marked for disconnection.
- Generated fragments are consistent with the probabilistic model that identifies 3185 unique small molecules at 0% FDR on 8 million GNPS spectra.
- No candidate cut edges of types other than N-C, O-C, and C-C enter the fragmentation graph.

## Limitations

- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules.
- Benzene rings and large resonant structures are naively treated as alternating single and double bonds.
- Lower accuracy in the low mass range (<600 Da), partly because fewer fragments are generated for small molecules.
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra.
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and is hard to extend.

## Evidence

- [full_text] molDiscovery constructs fragmentation graphs by first building a metabolite graph for a molecule, then applying a new efficient algorithm to find bridges and 2-cuts: [evidence span withheld — anchor status: not_found]
- [full_text] Run the Hopcroft-Tarjan algorithm to detect bridges and 2-cuts in the molecular graph: [evidence span withheld — anchor status: not_found]
- [intro] we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations"
- [readme] To test molDiscovery for academic users, please visit: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
