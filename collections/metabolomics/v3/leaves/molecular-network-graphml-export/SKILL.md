---
name: molecular-network-graphml-export
description: Use when a GNPS molecular networking job on deconvoluted EI GC-MS spectra
  (or MS2-based GNPS networks) has finished and you need the graphml network file
  to map Mass2Motifs/chemical classes and visualize in Cytoscape.
when_to_use_negative:
- Input is raw GC-MS data that has not been auto-deconvolved by MSHub — deconvolution
  must precede networking of EI spectra
- The analysis requires molecular-ion/precursor-mass-based networking (e.g. LC-MS/MS
  with known precursor m/z constraints) rather than EI spectral-similarity networking
- You need a network format other than graphml for downstream tools (the exported
  file is .graphml for Cytoscape/pyMolNetEnhancer)
tools:
- name: GNPS
  role: Runs the molecular networking workflow that computes pairwise cosine similarity
    between deconvoluted EI spectra and exports the network as .graphml
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSHub
  role: Produces the deconvoluted compound fragmentation spectra that become the nodes
    of the exported network
  repo: https://github.com/bittremieux/GNPS_GC
- name: pyMolNetEnhancer
  role: Consumes the exported GNPS .graphml to map chemical class and MS2LDA substructural
    information and re-emit an annotated .graphml for Cytoscape
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:336daf90487423ee8a5f19c1895f89f1c718c1788c6cfb4a55dfc7974ceaec13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# molecular-network-graphml-export

## Summary

Export GNPS molecular networks — with deconvoluted EI GC-MS fragmentation spectra as nodes and cosine-similarity scores as edge attributes — as a molecular_networks.graphml file for downstream chemical-class annotation (e.g. pyMolNetEnhancer) and Cytoscape visualization.

## When to use

Use when a GNPS molecular networking job on deconvoluted EI GC-MS spectra (or MS2-based GNPS networks) has finished and you need the graphml network file to map Mass2Motifs/chemical classes and visualize in Cytoscape.

## When NOT to use

- Input is raw GC-MS data that has not been auto-deconvolved by MSHub — deconvolution must precede networking of EI spectra
- The analysis requires molecular-ion/precursor-mass-based networking (e.g. LC-MS/MS with known precursor m/z constraints) rather than EI spectral-similarity networking
- You need a network format other than graphml for downstream tools (the exported file is .graphml for Cytoscape/pyMolNetEnhancer)

## Inputs

- GNPS molecular networking job (job ID) on deconvoluted EI GC-MS fragmentation spectra from MSHub/GNPS auto-deconvolution
- GNPS job output directory containing molecular_networks.graphml
- Optionally, MS2LDA job ID (from ms2lda.org) for motif mapping onto the network

## Outputs

- molecular_networks.graphml file with deconvoluted spectra as nodes and cosine-similarity scores as edge attributes
- Cytoscape-ready annotated .graphml (nodes/edges with Mass2Motif and chemical-class attributes)
- Optional Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv tables for separate Cytoscape import

## How to apply

Run the GNPS molecular networking workflow (CCMS-UCSD/GNPS_Workflows) on the deconvoluted compound fragmentation spectra from the MSHub/GNPS auto-deconvolution step; for EI data networking operates on spectral similarity of the fragmentation spectrum alone, without precursor/molecular-ion mass constraints, because EI spectra often lack the molecular ion. GNPS computes pairwise cosine similarity between all spectra, filters spectral pairs against the workflow's cosine-similarity and matched-peak criteria to keep high-confidence similarity edges, assembles per-dataset networks, and exports the result as molecular_networks.graphml. Download the .graphml output file from the GNPS job (using the job ID) for downstream processing: pyMolNetEnhancer can map chemical class and MS2LDA Mass2Motif information onto the network, and the annotated .graphml is then imported into Cytoscape for visualization (coloring edges by shared Mass2Motifs via the 'interaction' column, and nodes by TopSharedMotifs per molecular family/network component index).

## Related tools

- **GNPS** (Runs the molecular networking workflow that computes pairwise cosine similarity between deconvoluted EI spectra and exports the network as .graphml) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSHub** (Produces the deconvoluted compound fragmentation spectra that become the nodes of the exported network) — https://github.com/bittremieux/GNPS_GC
- **pyMolNetEnhancer** (Consumes the exported GNPS .graphml to map chemical class and MS2LDA substructural information and re-emit an annotated .graphml for Cytoscape) — https://github.com/madeleineernst/pyMolNetEnhancer

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- The exported molecular_networks.graphml is well-formed XML loadable into Cytoscape without import errors
- Nodes correspond to deconvoluted spectra (spectral IDs from the MSHub deconvolution MGF) and each edge carries a cosine-similarity score plus matched-peak attributes
- Retained edges satisfy the GNPS workflow's cosine-similarity and matched-peak filter criteria — filtered low-confidence pairs are absent
- After pyMolNetEnhancer annotation, node attributes include chemical classes / TopSharedMotifs and edge attributes include 'interaction' (shared Mass2Motifs) usable for Cytoscape discrete mapping
- Network components (molecular families / cluster indices) are consistent between the graphml and the CLUSTERID1/CLUSTERID2 columns of the edges table

## Limitations

- For EI GC-MS data the molecular ion is often absent, so networking rests purely on fragmentation-spectrum similarity and lacks precursor-mass edge constraints
- Depending on the size of the MS2LDA summary file, a server connection timeout may occur when fetching it; the summary can be downloaded manually from http://ms2lda.org/
- Probability (default 0.01) and overlap (default 0.3) thresholds for Mass2Motif mapping are user-settable and affect which annotations appear in the graphml

## Evidence

- [readme] To visualize results import the .graphml output file into Cytoscape: [evidence span withheld — anchor status: external_doc]
- [readme] Create a molecular network through the GNPS platform: [evidence span withheld — anchor status: not_found]
- [readme] Depending on the size of this file, a server connection timeout may occur: [evidence span withheld — anchor status: external_doc]
- [readme] import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape: [evidence span withheld — anchor status: external_doc]
