---
name: deconvoluted-spectrum-network-edge-construction
description: Use when EI GC-MS data has been deconvoluted into individual compound
  fragmentation spectra (e.g. via MSHub/GNPS auto-deconvolution) and spectrally related
  compounds must be grouped into molecular networks without precursor-ion masses.
when_to_use_negative:
- Input is LC-MS/MS data where precursor ion masses are known — the standard precursor-based
  GNPS networking applies instead of this EI-specific configuration
- Spectra have not yet been deconvoluted from co-eluting GC-MS peaks — run MSHub auto-deconvolution
  first, since networking operates on individual compound fragmentation spectra
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Hosts the molecular networking workflow that computes pairwise cosine similarity
    between deconvoluted EI spectra and assembles filtered edges into networks
- name: GNPS_Workflows
  role: Publicly released workflow scripts (CCMS-UCSD/GNPS_Workflows) run in gas-chromatography
    configuration on EI spectra without precursor mass constraints
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSHub
  role: Produces the deconvoluted compound fragmentation spectra that serve as the
    node-level input to EI molecular networking
- name: pyMolNetEnhancer
  role: Downstream option to map chemical class and substructure information onto
    the GNPS molecular network GraphML output
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:ba29e11b58b49f3983f7b3fda8c2ac0a2417e41c6221014dbabc93749dfd1f88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - GNPS_Workflows
  - MSHub
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# deconvoluted-spectrum-network-edge-construction

## Summary

Constructs GNPS molecular networks for EI GC-MS data by computing pairwise cosine similarity between deconvoluted compound fragmentation spectra, filtering edges on cosine-similarity and matched-peak criteria, and exporting the network as GraphML — all without precursor or molecular-ion masses, which are often absent in EI spectra.

## When to use

Use when EI GC-MS data has been deconvoluted into individual compound fragmentation spectra (e.g. via MSHub/GNPS auto-deconvolution) and spectrally related compounds must be grouped into molecular networks without precursor-ion masses.

## When NOT to use

- Input is LC-MS/MS data where precursor ion masses are known — the standard precursor-based GNPS networking applies instead of this EI-specific configuration
- Spectra have not yet been deconvoluted from co-eluting GC-MS peaks — run MSHub auto-deconvolution first, since networking operates on individual compound fragmentation spectra

## Inputs

- Deconvoluted compound fragmentation spectra (EI GC-MS) from the MSHub/GNPS auto-deconvolution step
- GNPS molecular networking workflow configured for gas chromatography (CCMS-UCSD/GNPS_Workflows)

## Outputs

- Per-dataset molecular networks with deconvoluted spectra as nodes and cosine-similarity scores as edge attributes
- molecular_networks.graphml for downstream annotation and visualization

## How to apply

Load the deconvoluted compound fragmentation spectra produced by the MSHub/GNPS auto-deconvolution step as the input spectral file for networking. Run the GNPS molecular networking workflow (CCMS-UCSD/GNPS_Workflows) in its gas-chromatography configuration, which operates on EI spectra without precursor/molecular-ion mass constraints, unlike the LC-MS/MS workflow where the precursor mass is known. Compute pairwise cosine similarity between all deconvoluted EI fragmentation spectra within the dataset, then filter spectral pairs against the workflow's cosine-similarity and matched-peak criteria to retain only high-confidence similarity edges. Assemble the retained edges into per-dataset molecular networks, with deconvoluted spectra as nodes and similarity scores as edge attributes, and export the structure as molecular_networks.graphml for downstream annotation and visualization (e.g. in Cytoscape).

## Related tools

- **GNPS** (Hosts the molecular networking workflow that computes pairwise cosine similarity between deconvoluted EI spectra and assembles filtered edges into networks)
- **GNPS_Workflows** (Publicly released workflow scripts (CCMS-UCSD/GNPS_Workflows) run in gas-chromatography configuration on EI spectra without precursor mass constraints) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSHub** (Produces the deconvoluted compound fragmentation spectra that serve as the node-level input to EI molecular networking)
- **pyMolNetEnhancer** (Downstream option to map chemical class and substructure information onto the GNPS molecular network GraphML output) — https://github.com/madeleineernst/pyMolNetEnhancer

## Evaluation signals

- Edge list contains only spectral pairs passing the workflow's cosine-similarity and matched-peak criteria; unfiltered pairwise similarities should not appear
- Nodes in molecular_networks.graphml correspond one-to-one to deconvoluted EI compound fragmentation spectra, not to raw MS/MS scans with precursor masses
- Edges carry cosine-similarity scores as edge attributes that can be inspected after GraphML import
- GraphML loads in Cytoscape and renders connected components ('molecular families') grouping spectrally similar EI spectra
- No precursor/molecular-ion mass field is required or used in the networking — networks form purely from fragmentation-spectrum similarity

## Limitations

- The molecular ion is often absent in EI spectra, so unlike LC-MS/MS networking no precursor mass anchors the network — grouping relies entirely on fragmentation-spectrum cosine similarity
- Edge quality depends on the upstream MSHub deconvolution: poorly deconvoluted fragmentation patterns propagate into unreliable network edges
- Specific cosine-similarity and matched-peak threshold values are set in the workflow configuration and are not stated in the article text

## Evidence

- [readme] GraphML output is imported into Cytoscape for visualization: [evidence span withheld — anchor status: not_found]
