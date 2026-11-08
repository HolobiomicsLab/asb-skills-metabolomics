---
name: ei-cosine-similarity-spectral-matching
description: Use when networking EI GC-MS fragmentation spectra (e.g. MSHub/GNPS deconvoluted
  compound spectra) where the molecular ion is absent, so relationships must be inferred
  from spectral similarity alone.
when_to_use_negative:
- When precursor/molecular-ion mass is available (e.g. LC-MS/MS data), where classical
  GNPS precursor-mass-constrained networking is appropriate instead
- When the input spectra have not been deconvoluted — overlapping, co-eluting raw
  GC-MS spectra should first go through MSHub auto-deconvolution
- When the goal is annotation/chemical-class mapping of an existing network (e.g.
  pyMolNetEnhancer) rather than constructing the network itself
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Platform whose molecular networking workflow performs cosine-similarity networking
    of the deconvoluted EI spectra
- name: GNPS_Workflows
  role: Publicly released GNPS ProteoSAFe workflow scripts, run in gas chromatography
    configuration without precursor-mass constraints
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSHub
  role: Produces the auto-deconvoluted EI fragmentation spectra that serve as the
    input spectral file for networking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:8b2bcea945eda4e31032fd11f88571f539a1b844b7aff03ab0397bda19393212
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - GNPS_Workflows
  - MSHub
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ei-cosine-similarity-spectral-matching

## Summary

Constructs GNPS molecular networks for EI GC-MS data by computing pairwise cosine similarity between deconvoluted EI fragmentation spectra, without using precursor or molecular-ion mass. This is how spectrally related compounds are linked into molecular families when the molecular ion is absent in EI spectra.

## When to use

Use when networking EI GC-MS fragmentation spectra (e.g. MSHub/GNPS deconvoluted compound spectra) where the molecular ion is absent, so relationships must be inferred from spectral similarity alone.

## When NOT to use

- When precursor/molecular-ion mass is available (e.g. LC-MS/MS data), where classical GNPS precursor-mass-constrained networking is appropriate instead
- When the input spectra have not been deconvoluted — overlapping, co-eluting raw GC-MS spectra should first go through MSHub auto-deconvolution
- When the goal is annotation/chemical-class mapping of an existing network (e.g. pyMolNetEnhancer) rather than constructing the network itself

## Inputs

- Deconvoluted compound fragmentation spectra from the MSHub/GNPS auto-deconvolution step (input spectral file for networking)
- EI GC-MS fragmentation spectra without precursor/molecular-ion mass information

## Outputs

- Per-dataset molecular networks with deconvoluted spectra as nodes and similarity scores as edge attributes
- molecular_networks.graphml for downstream annotation and visualization

## How to apply

Load the deconvoluted compound fragmentation spectra produced by the MSHub/GNPS auto-deconvolution step as the input spectral file for networking. Set up the GNPS molecular networking workflow (publicly released as the CCMS-UCSD/GNPS_Workflows scripts) in its gas chromatography configuration, operating on EI spectra without precursor/molecular-ion mass constraints. Compute pairwise cosine similarity between all deconvoluted EI fragmentation spectra within each dataset and filter spectral pairs against the workflow's cosine-similarity and matched-peak criteria to retain only high-confidence similarity relationships. Assemble retained similarity edges into per-dataset molecular networks, with deconvoluted spectra as nodes and similarity scores as edge attributes, and export the network structure as molecular_networks.graphml for downstream annotation and visualization.

## Related tools

- **GNPS** (Platform whose molecular networking workflow performs cosine-similarity networking of the deconvoluted EI spectra)
- **GNPS_Workflows** (Publicly released GNPS ProteoSAFe workflow scripts, run in gas chromatography configuration without precursor-mass constraints) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSHub** (Produces the auto-deconvoluted EI fragmentation spectra that serve as the input spectral file for networking)

## Evaluation signals

- Retained spectral pairs meet the workflow's cosine-similarity and matched-peak criteria (high-confidence edges only)
- Output molecular_networks.graphml parses and encodes deconvoluted spectra as nodes with similarity scores as edge attributes
- Edges are computed within each dataset only, from spectral similarity of EI fragmentation patterns rather than precursor mass
- Network components group spectrally related compound pairs, consistent with reproducible fragmentation patterns quantified by the balance score

## Limitations

- Because the molecular ion is often absent in EI spectra, networks are built purely on spectral similarity without any precursor-mass constraint, so edges reflect fragmentation-pattern resemblance rather than mass relationships
- Network quality depends on the upstream MSHub/GNPS auto-deconvolution (unsupervised non-negative matrix factorization) correctly separating co-eluting compound spectra
- Fast Fourier Transform-based strategy is needed to overcome scalability limitations of the deconvolution on large datasets

## Evidence

- [other] For EI data, GNPS creates molecular networks through spectral similarity of the deconvoluted fragmentation spectrum without considering the molecular ion.: [evidence span withheld — anchor status: not_found]
- [readme] The workflows repository contains the GNPS Workflows running on GNPS.: [evidence span withheld — anchor status: external_doc]
