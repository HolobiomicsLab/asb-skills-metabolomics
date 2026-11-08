---
name: gnps-task-file-resolution
description: 'Use when input contains one or more mzspec: USI strings (e.g. mzspec:GNPS:TASK-...,
  mzspec:MSV..., mzspec:MTBLS...) and the goal is to obtain the resolved remote URL
  and the referenced spectrum file for further analysis or provenance capture.'
when_to_use_negative:
- The data is already a local mzXML/mzML file — no USI resolution is needed, proceed
  directly to visualization or peak processing.
- 'The identifier is a raw accession without the mzspec: USI syntax (e.g. a bare MSV
  or MTBLS accession) — the USI resolution logic requires the full mzspec format including
  dataset and file/scan specifier.'
- The target repository is not GNPS/MassIVE or MetaboLights and no _resolve_usi_remotelink
  extension has been implemented for it — per the README, new data sources require
  code changes in download.py.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: GNPS_LCMSDashboard
  role: Implements the USI-to-remote-URL resolution logic in download.py (_resolve_usi_remotelink),
    used to fetch the spectrum files behind each USI
  repo: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
- name: Universal Spectrum Identifier (USI)
  role: Identifier format that encodes repository prefix, dataset identifier, file
    path, and scan specifier for cross-repository data retrieval
- name: mzXML format (centroided MS data files)
  role: Spectrum file format referenced by the GNPS TASK example USI (0518_s_BuOH.mzXML)
- name: mzML format (open MS data format)
  role: Spectrum file format referenced by the MetaboLights example USI (QC07.mzML)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01339-5
    title: 'GNPS Dashboard: collaborative exploration of mass spectrometry data in
      the web browser'
  source_content_hash: sha256:c4d42fc7074043f70399139ec398c00c4b10b6472ebfead9a39b94d8e08e0fc0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
  repo_url: https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
  tools:
  - GNPS_LCMSDashboard
  - Universal Spectrum Identifier (USI)
  - mzML format (open MS data format)
  - mzXML format (centroided MS data files)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gnps-task-file-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves mzspec-prefixed Universal Spectrum Identifiers (USIs) referencing GNPS TASK outputs, MassIVE MSV accessions, or MetaboLights MTBLS accessions to their underlying LC-MS run files (mzXML/mzML), as implemented in the GNPS LC-MS Dashboard's USI resolution logic.

## When to use

Use when input contains one or more mzspec: USI strings (e.g. mzspec:GNPS:TASK-..., mzspec:MSV..., mzspec:MTBLS...) and the goal is to obtain the resolved remote URL and the referenced spectrum file for further analysis or provenance capture.

## When NOT to use

- The data is already a local mzXML/mzML file — no USI resolution is needed, proceed directly to visualization or peak processing.
- The identifier is a raw accession without the mzspec: USI syntax (e.g. a bare MSV or MTBLS accession) — the USI resolution logic requires the full mzspec format including dataset and file/scan specifier.
- The target repository is not GNPS/MassIVE or MetaboLights and no _resolve_usi_remotelink extension has been implemented for it — per the README, new data sources require code changes in download.py.

## Inputs

- mzspec-prefixed USI strings, e.g. mzspec:GNPS:TASK-d93bdbb5cdda40e48975e6e18a45c3ce-f.mwang87/data/Yao_Streptomyces/roseosporus/0518_s_BuOH.mzXML:scan:171
- MassIVE accession USIs such as mzspec:MSV000084951:AH22
- MetaboLights accession USIs such as mzspec:MTBLS1124:QC07.mzML

## Outputs

- Per-USI resolution record containing resolved remote URL, repository prefix, file format (mzXML or mzML), and original USI string (e.g. usi_resolved_runs.json)
- Retrieved spectrum files referenced by the resolved USIs

## How to apply

Parse each USI into its repository prefix (GNPS, MassIVE MSV, or MetaboLights MTBLS), dataset identifier, and optional file path plus scan specifier. Dispatch resolution by prefix: GNPS TASK USIs resolve through the GNPS task data path, MSV accessions through MassIVE download endpoints, and MTBLS accessions through MetaboLights download endpoints — the exact dispatch logic lives in the GNPS_LCMSDashboard download.py module's _resolve_usi_remotelink function, which the README names as the extension point for new data sources. Retrieve the referenced spectrum file (e.g. 0518_s_BuOH.mzXML for the GNPS task USI, QC07.mzML for MTBLS1124) and record resolution status, resolved remote URL, file format, and the original USI string per entry. Validate by confirming each record carries a resolved URL, a recognized format (mzXML or mzML), and the unaltered input USI.

## Related tools

- **GNPS_LCMSDashboard** (Implements the USI-to-remote-URL resolution logic in download.py (_resolve_usi_remotelink), used to fetch the spectrum files behind each USI) — https://github.com/Wang-Bioinformatics-Lab/GNPS_LCMSDashboard
- **Universal Spectrum Identifier (USI)** (Identifier format that encodes repository prefix, dataset identifier, file path, and scan specifier for cross-repository data retrieval)
- **mzXML format (centroided MS data files)** (Spectrum file format referenced by the GNPS TASK example USI (0518_s_BuOH.mzXML))
- **mzML format (open MS data format)** (Spectrum file format referenced by the MetaboLights example USI (QC07.mzML))

## Examples

```
https://gnps-lcms.ucsd.edu/?usi=mzspec%3AGNPS%3ATASK-d93bdbb5cdda40e48975e6e18a45c3ce-f.mwang87%2Fdata%2FYao_Streptomyces%2Froseosporus%2F0518_s_BuOH.mzXML%3Ascan%3A171&xicmz=841.3170166%3B842.3170166
```

## Evaluation signals

- All three example USIs (GNPS TASK, MSV000084951, MTBLS1124) resolve to a reachable remote URL without errors.
- Each output record contains a non-empty resolved URL, a recognized file format field limited to mzXML or mzML, and the original USI string preserved verbatim.
- The repository prefix parsed from each USI (GNPS, MSV, MTBLS) matches the endpoint actually used for resolution.
- Retrieved files open as valid centroided LC-MS run data (parseable as mzXML or mzML with a minimal run summary extractable).

## Limitations

- Resolution is limited to supported repositories; supporting a new data source requires implementing a USI specification and updating _resolve_usi_remotelink in download.py.
- GNPS TASK USIs depend on task data availability on the GNPS server; task-related data may not persist indefinitely.
- The USI may omit a file path or scan specifier (e.g. mzspec:MSV000084951:AH22), in which case the resolved resource is dataset/file-level rather than a specific scan.

## Evidence

- [readme] Update the code in ```download.py```, specifically in ```_resolve_usi_remotelink``` to implement how to get the remote URL for your new USI: "Update the code in ```download.py```, specifically in ```_resolve_usi_remotelink``` to implement how to get the remote URL for your new USI"
- [readme] USI Specification that denotes what the resource is and how to get data: "1. USI Specification that denotes what the resource is and how to get data"
- [intro] mzspec:GNPS:TASK-d93bdbb5cdda40e48975e6e18a45c3ce-f.mwang87/data/Yao_Streptomyces/roseosporus/0518_s_BuOH.mzXML:scan:171: "mzspec:GNPS:TASK-d93bdbb5cdda40e48975e6e18a45c3ce-f.mwang87/data/Yao_Streptomyces/roseosporus/0518_s_BuOH.mzXML:scan:171"
- [intro] supporting datasets from GNPS/MassIVE and MetaboLights repositories via USI references: [evidence span withheld — anchor status: not_found]
