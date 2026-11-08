---
name: gnps-workflow-configuration
description: Use when building molecular networks from deconvoluted EI GC-MS fragmentation
  spectra in GNPS, where the molecular ion is often absent and LC-MS/MS precursor-mass-based
  networking cannot be applied.
when_to_use_negative:
- Input is LC-MS/MS data where precursor/molecular-ion masses are known — standard
  GNPS precursor-based molecular networking applies instead of the EI GC-MS configuration.
- Input is raw, co-eluted GC-MS data that has not yet been through MSHub/GNPS auto-deconvolution
  — spectral deconvolution must be performed first before spectra can be networked.
- Goal is deconvolution quality control itself (NMF fragmentation patterns, balance-score
  reproducibility) rather than constructing the network from already-deconvoluted
  spectra.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MSHub
  role: Performs auto-deconvolution of GC-MS compound fragmentation patterns via unsupervised
    non-negative matrix factorization; its deconvoluted spectra are the networking
    input
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Hosts and executes the molecular networking workflow for GC-MS data (EI cosine-similarity
    networking without precursor mass)
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: GNPS_Workflows
  role: Public repository of the ProteoSAFe workflow definitions, used in the gas
    chromatography configuration for EI spectra
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: pyMolNetEnhancer
  role: Optional downstream step that maps chemical class and substructure information
    onto the GNPS molecular network graphml output
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:3af93fe948298f7a92476c7f07bed903e635a43cff2038af5b858542e43a47b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - GNPS_Workflows
  - MSHub
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gnps-workflow-configuration

## Summary

Configure and run the GNPS molecular networking workflow (CCMS-UCSD/GNPS_Workflows) in its gas-chromatography configuration, which networks deconvoluted EI GC-MS fragmentation spectra by cosine spectral similarity instead of the precursor/molecular-ion mass used in LC-MS/MS networking.

## When to use

Use when building molecular networks from deconvoluted EI GC-MS fragmentation spectra in GNPS, where the molecular ion is often absent and LC-MS/MS precursor-mass-based networking cannot be applied.

## When NOT to use

- Input is LC-MS/MS data where precursor/molecular-ion masses are known — standard GNPS precursor-based molecular networking applies instead of the EI GC-MS configuration.
- Input is raw, co-eluted GC-MS data that has not yet been through MSHub/GNPS auto-deconvolution — spectral deconvolution must be performed first before spectra can be networked.
- Goal is deconvolution quality control itself (NMF fragmentation patterns, balance-score reproducibility) rather than constructing the network from already-deconvoluted spectra.

## Inputs

- Deconvoluted compound fragmentation spectra from the MSHub/GNPS auto-deconvolution step (input spectral file for networking)
- GNPS workflow parameters configured for gas chromatography (EI mode, no precursor/molecular-ion mass constraints)

## Outputs

- Per-dataset molecular networks with deconvoluted EI spectra as nodes and cosine-similarity scores as edge attributes
- Pairwise spectral similarity edges retained after cosine-similarity and matched-peak filtering
- molecular_networks.graphml export for downstream annotation and visualization

## How to apply

Load the deconvoluted compound fragmentation spectra produced by MSHub/GNPS auto-deconvolution (unsupervised non-negative matrix factorization, balance-score-filtered) as the input spectral file. Configure the GNPS ProteoSAFe workflow from the CCMS-UCSD/GNPS_Workflows repository in its gas chromatography configuration, operating on EI spectra without precursor/molecular-ion mass constraints. The workflow computes pairwise cosine similarity between all deconvoluted EI fragmentation spectra within each dataset, then filters spectral pairs against cosine-similarity and matched-peak criteria to retain only high-confidence similarity relationships. Retained edges are assembled into per-dataset molecular networks with deconvoluted spectra as nodes and similarity scores as edge attributes, and exported as molecular_networks.graphml for downstream annotation (e.g., pyMolNetEnhancer chemical-class mapping) and Cytoscape visualization.

## Related tools

- **MSHub** (Performs auto-deconvolution of GC-MS compound fragmentation patterns via unsupervised non-negative matrix factorization; its deconvoluted spectra are the networking input) — github.com/bittremieux/GNPS_GC
- **GNPS** (Hosts and executes the molecular networking workflow for GC-MS data (EI cosine-similarity networking without precursor mass)) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **GNPS_Workflows** (Public repository of the ProteoSAFe workflow definitions, used in the gas chromatography configuration for EI spectra) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **pyMolNetEnhancer** (Optional downstream step that maps chemical class and substructure information onto the GNPS molecular network graphml output) — https://github.com/madeleineernst/pyMolNetEnhancer

## Evaluation signals

- The workflow produces a molecular_networks.graphml file whose nodes are deconvoluted EI spectra rather than precursor-ion features.
- Network edges carry pairwise cosine-similarity scores that passed the workflow's cosine-similarity and matched-peak filtering criteria.
- Network construction succeeds despite absent molecular ions — no precursor-mass window parameters should be required for EI spectra.
- Spectrally related compound pairs within each dataset should be connected, while dissimilar deconvoluted spectra are absent as edges.
- The graphml can be loaded into Cytoscape (and optionally annotated with chemical classes via pyMolNetEnhancer) without schema errors.

## Limitations

- Because EI spectra typically lack the molecular ion, networking relies entirely on fragmentation-spectrum similarity; compounds with similar EI fragmentation but different identities may be joined.
- Input quality depends on the upstream MSHub auto-deconvolution (NMF) and the balance score quantifying fragmentation-pattern reproducibility across samples.
- Scalability of deconvolution was a constraint, addressed with a Fast Fourier Transform-based strategy; very large datasets may still strain workflow resources.

## Evidence

- [readme] Workflow repository provenance: [evidence span withheld — anchor status: external_doc]
- [readme] Downstream network annotation path: [evidence span withheld — anchor status: external_doc]
