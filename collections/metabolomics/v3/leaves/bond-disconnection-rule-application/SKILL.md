---
name: bond-disconnection-rule-application
description: Use when generating candidate fragmentation graphs for small molecules
  (SMILES/SDF, up to ~2000 Da) to match against tandem mass spectra during in silico
  database search, especially when rule-based enumeration is too slow for heavy molecules.
when_to_use_negative:
- The query molecule is peptidic and you need a p-value — molDiscovery's MCMC p-value
  approach is limited to peptidic molecules, and extending it to general molecules
  'is challenging as it involves generation of random molecular structures'
- 'The molecule is very small (<400 Da): the article notes existing methods ''do not
  perform well for super small molecules (<400 Da)'', and molDiscovery itself reports
  lower accuracy below 600 Da because ''fewer fragments are generated for molecules
  in this range'''
- Accurate modeling of resonant structures or rearrangement rules is required — molDiscovery
  'naively treats benzene ring or large resonant structures as alternate single bonds
  and double bonds' and covers only a limited number of bond types and rearrangement
  rules
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: molDiscovery
  role: 'Implements the fragmentation graph construction: builds the metabolite graph,
    disconnects N-C/O-C/C-C bonds, finds bridges and 2-cuts, and generates fragments
    for spectral database search'
  repo: github.com/mohimanilab/molDiscovery
- name: MetFrag
  role: Contrasting rule-based in silico fragmentation approach using weighted peak
    count and bond dissociation energy; baseline the bridge/2-cut algorithm improves
    upon
- name: CFM-ID
  role: Machine-learning spectral prediction alternative (stochastic Markov process)
    that performs poorly for small molecules; motivates the efficient fragmentation
    algorithm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:2cb0767b804c79dc862f6793c76511ead1956fe98e2407d55ab395b9fb52cd4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MetFrag
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bond-disconnection-rule-application

## Summary

Generates in silico mass spectrometry fragments for small molecules by disconnecting candidate bonds (bridges and 2-cuts) in a molecular graph, as implemented in molDiscovery to improve efficiency and accuracy of small molecule identification from tandem mass spectra.

## When to use

Use when generating candidate fragmentation graphs for small molecules (SMILES/SDF, up to ~2000 Da) to match against tandem mass spectra during in silico database search, especially when rule-based enumeration is too slow for heavy molecules.

## When NOT to use

- The query molecule is peptidic and you need a p-value — molDiscovery's MCMC p-value approach is limited to peptidic molecules, and extending it to general molecules 'is challenging as it involves generation of random molecular structures'
- The molecule is very small (<400 Da): the article notes existing methods 'do not perform well for super small molecules (<400 Da)', and molDiscovery itself reports lower accuracy below 600 Da because 'fewer fragments are generated for molecules in this range'
- Accurate modeling of resonant structures or rearrangement rules is required — molDiscovery 'naively treats benzene ring or large resonant structures as alternate single bonds and double bonds' and covers only a limited number of bond types and rearrangement rules

## Inputs

- Molecular structures in SMILES or SDF format from a structure database (e.g., PubChem, Dictionary of Natural Products)
- Tandem mass spectra (e.g., from the GNPS repository) for downstream spectral matching
- Graph representation of each molecule (atoms as nodes, bonds as edges)

## Outputs

- Fragmentation graph with fragments from single-bond cuts (bridges) and double-bond cuts (2-cuts)
- Binary indicator vectors encoding each fragment as a subset of the molecule's atoms
- Depth-n fragment vectors (bitwise intersections of parent fragment vectors) used for spectral matching

## How to apply

Load molecular structures (SMILES/SDF) from a structure database such as PubChem or the Dictionary of Natural Products, representing each molecule as a graph with atoms as nodes and bonds as edges. Disconnect all N-C, O-C, and C-C bonds to define candidate fragmentation edges. Run the Hopcroft-Tarjan algorithm to find bridges and 2-cuts in the molecular graph, identifying single- and double-bond cuts that yield fragments. Encode each single-bond-cut fragment as a binary indicator vector over the molecule's atoms, then build depth-n fragments as the bitwise intersection of parent fragment vectors for all combinations of n cut edges. These fragments feed molDiscovery's probabilistic model for scoring molecule-spectrum pairs in database search against datasets like the 8 million GNPS tandem mass spectra.

## Related tools

- **molDiscovery** (Implements the fragmentation graph construction: builds the metabolite graph, disconnects N-C/O-C/C-C bonds, finds bridges and 2-cuts, and generates fragments for spectral database search) — github.com/mohimanilab/molDiscovery
- **MetFrag** (Contrasting rule-based in silico fragmentation approach using weighted peak count and bond dissociation energy; baseline the bridge/2-cut algorithm improves upon)
- **CFM-ID** (Machine-learning spectral prediction alternative (stochastic Markov process) that performs poorly for small molecules; motivates the efficient fragmentation algorithm)

## Evaluation signals

- Every depth-n fragment must equal the bitwise intersection of valid parent fragment indicator vectors
- Pipeline completes on test molecules spanning the 0–2000 Da mass range
- Fragments are only generated by cutting bridges and 2-cuts after disconnecting all N-C, O-C, and C-C bonds
- End-to-end check: database search of GNPS spectra should show improved unique molecule identifications (molDiscovery identified 3185 unique small molecules at 0% FDR, six times more than previous methods)

## Limitations

- Lower accuracy below 600 Da because fewer fragments are generated for molecules in this mass range
- Only a limited number of bond types and rearrangement rules are covered; local/global structural features (e.g., moieties) are ignored
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- No p-value computation for general molecules; the MCMC p-value approach applies only to peptidic molecules

## Evidence

- [other] molDiscovery constructs fragmentation graphs by first building a metabolite graph for a molecule, then applying a new efficient algorithm to find bridges and 2-cuts: [evidence span withheld — anchor status: not_found]
- [intro] we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations and (ii) developing a probabilistic model: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations and (ii) developing a probabilistic model"
- [discussion] This is partially because fewer fragments are generated for molecules in this range.: "This is partially because fewer fragments are generated for molecules in this range."
- [discussion] molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds: "molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds"
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
