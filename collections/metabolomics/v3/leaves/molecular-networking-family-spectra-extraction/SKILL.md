---
name: molecular-networking-family-spectra-extraction
description: Use when an analysis needs the complete set of MS/MS spectra belonging
  to one molecular networking component (Task ID + component number, or a list of
  USIs) as input to multiple spectral alignment rather than single-spectrum inspection.
when_to_use_negative:
- Input is already an aligned peak set or a feature quantification table rather than
  tandem MS spectra of a network component
- Only a single spectrum needs inspection (use Metabolomics Spectrum Resolver or ModiFinder
  instead of family-level extraction)
- The spectra do not originate from a molecular networking task or lack valid USI/task
  identifiers
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS2
  role: Source of molecular networking tasks; MMSA fetches components by task ID and
    component number
- name: metabolomics resolver
  role: Resolves USIs from public data sources (MassIVE/GNPS, Metabolomics Workbench,
    PRIDE, GNPS2) into spectra
- name: MMSA
  role: Web application that accepts Task ID/USI/FBMN input, processes the family,
    and saves alignment JSON
  repo: github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Builds the minimum spanning tree over the extracted family spectra, edge-weighted
    by 1 minus modified cosine similarity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:4b1c4925b8c40579083bd3188cf6a6aa35b2337759df5591035de6334dfc57a2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Python Dash
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-networking-family-spectra-extraction

## Summary

Retrieves all tandem MS spectra of a molecular network component (e.g., the rhamnolipid family from the seven P. aeruginosa strain LC-MS/MS dataset) from GNPS2 tasks or public repositories via USIs, so the family can be jointly aligned and visualized in MMSA.

## When to use

Use when an analysis needs the complete set of MS/MS spectra belonging to one molecular networking component (Task ID + component number, or a list of USIs) as input to multiple spectral alignment rather than single-spectrum inspection.

## When NOT to use

- Input is already an aligned peak set or a feature quantification table rather than tandem MS spectra of a network component
- Only a single spectrum needs inspection (use Metabolomics Spectrum Resolver or ModiFinder instead of family-level extraction)
- The spectra do not originate from a molecular networking task or lack valid USI/task identifiers

## Inputs

- GNPS2 task ID and component number (or FBMN task ID with component numbers)
- List of Universal Spectrum Identifiers (one per line), format mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
- Molecular network component from a GNPS2 feature-based molecular networking job (specs_ms.mgf clustering output)

## Outputs

- Persisted alignment JSON for the molecular family component (via 'Process and Save JSON')
- Set of MS/MS spectra for the component (e.g., the RL family spectra from seven P. aeruginosa strains) ready for MST-based multiple spectral alignment
- Interactive peak alignment visualization with SVG export

## How to apply

Source the molecular network from a GNPS2 task by entering a task ID and component number, or supply Universal Spectrum Identifiers of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, one per line, resolving them from public repositories such as MassIVE/GNPS, Metabolomics Workbench, PRIDE, and GNPS2 through the metabolomics resolver. Trigger 'Process and Save JSON' in the MMSA interface to fetch the specs_ms.mgf spectra for the component and persist the alignment data. The extracted family (e.g., the rhamnolipid component containing C10- and C12-HAA mono-/di-RLs) then serves as input to the MST-based alignment: preprocess with peak filtering (top 6 most intense peaks within ±25 m/z, excluding peaks within ±17 m/z of the precursor), square root intensity transformation, and normalization before NetworkX 3.3 MST construction. Decide between Task ID/Component, USI, and FBMN input modes based on whether the component exists in an existing FBMN task or only as individual spectra.

## Related tools

- **GNPS2** (Source of molecular networking tasks; MMSA fetches components by task ID and component number)
- **metabolomics resolver** (Resolves USIs from public data sources (MassIVE/GNPS, Metabolomics Workbench, PRIDE, GNPS2) into spectra)
- **MMSA** (Web application that accepts Task ID/USI/FBMN input, processes the family, and saves alignment JSON) — github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Builds the minimum spanning tree over the extracted family spectra, edge-weighted by 1 minus modified cosine similarity)

## Examples

```
mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
```

## Evaluation signals

- The fetched component for the RL family yields spectra whose consensus peak sets include m/z 135.017 in all RL spectra, and m/z 211.130 and m/z 357.189 in the same ten spectra
- Every USI resolves without error and follows the mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> format
- Alignment JSON is written to the temp/sets path after 'Process and Save JSON' completes without fetch errors
- The number of spectra extracted matches the component size reported by the GNPS2 task (e.g., the ten-spectrum C10 RL subset within the family)

## Limitations

- Extraction depends on an existing molecular networking task; no standalone repo URL or Zenodo archive for the software is provided, only the local directory NetworkFamily_MultipleAlignment_Website
- Extraction alone does not guarantee complete downstream alignment — shifted peaks such as the C12 HAA (m/z 239.162) and rhamnose-C12 (m/z 385.219) were not captured by MMSA's greedy alignment
- Only spectra addressable via USI-supporting public repositories (MassIVE/GNPS, Metabolomics Workbench, PRIDE, GNPS2) can be retrieved

## Evidence

- [results] input USIs via metabolomics resolver: [evidence span withheld — anchor status: not_found]
