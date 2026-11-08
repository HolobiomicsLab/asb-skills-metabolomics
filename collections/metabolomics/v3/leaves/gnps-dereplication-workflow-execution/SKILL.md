---
name: gnps-dereplication-workflow-execution
description: Use when mzML LC-MS/MS runs have been processed by Optimus and detected
  features need putative compound identities via MS/MS spectral library matching at
  GNPS, to be attached by mz-RT matching.
when_to_use_negative:
- Features have already been annotated or a validated compound-annotation table exists
  — re-running GNPS dereplication would duplicate work
- The goal is level-1 identified compounds with MS/MS spectral validation — Optimus
  only provides level-2 putative annotations and does not perform MS/MS validation
- Data is direct-infusion MS without LC-MS/MS spectra — no MS/MS spectra can be matched
  for dereplication
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow that detects/aligns LC-MS features with OpenMS, exports feature
    spectra as MGF/MAT, and attaches GNPS-derived annotations via 'Read mz-RT list'
    and 'Match features by mz-RT'
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the LC-MS feature detection and quantification algorithms used inside
    Optimus to pair features with their MS/MS spectra
  repo: https://github.com/MolecularCartography/Optimus
- name: KNIME
  role: Workflow management platform hosting the Optimus pipeline including the 'Prepare
    results', 'Read mz-RT list' and 'Match features by mz-RT' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: GNPS
  role: Receives uploaded mzML files and runs Dereplication MS/MS matching against
    public spectral libraries, producing the annotation source exported as .tsv
- name: MS-FINDER
  role: Alternative downstream tool consuming MAT-format spectral exports of detected
    features
- name: Sirius
  role: Alternative downstream tool for In-Silico prediction consuming Optimus's spectral
    exports
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:5f862396466bfa700d649ce3dfd870d06a831e71ea64379755624fdfbcbd8495
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - GNPS
  - KNIME
  - MS-FINDER
  - OpenMS
  - Optimus
  - Sirius
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gnps-dereplication-workflow-execution

## Summary

Runs GNPS dereplication on LC-MS/MS runs exported from Optimus to obtain level-2 (Metabolomics Standards Initiative) putative molecular annotations, which are fed back into Optimus via mz-RT matching against a molecules-of-interest list.

## When to use

Use when mzML LC-MS/MS runs have been processed by Optimus and detected features need putative compound identities via MS/MS spectral library matching at GNPS, to be attached by mz-RT matching.

## When NOT to use

- Features have already been annotated or a validated compound-annotation table exists — re-running GNPS dereplication would duplicate work
- The goal is level-1 identified compounds with MS/MS spectral validation — Optimus only provides level-2 putative annotations and does not perform MS/MS validation
- Data is direct-infusion MS without LC-MS/MS spectra — no MS/MS spectra can be matched for dereplication

## Inputs

- mzML LC-MS/MS runs from a maXis II QTOF (or comparable instrument)
- Optimus KNIME workflow with OpenMS-based feature detection and 'Prepare results' node
- GNPS account for uploading mzML files and running Dereplication jobs
- Filtered GNPS .tsv or a manually created 3-column CSV (m/z, RT in seconds, Compound name)

## Outputs

- MGF (and optionally MAT) spectral export files of detected features
- Filtered GNPS dereplication .tsv with polarity and 20 p.p.m. precursor tolerance applied
- annotated_features.csv: the feature quantification matrix with an added compound-annotation field for matched features

## How to apply

Load mzML runs into the Optimus KNIME workflow and detect/align features with OpenMS, setting 'Half of MS/MS isolation window (Da)' so each feature is paired with its MS/MS spectrum. Enable MS/MS export in the 'Prepare results' node by checking 'Save as MGF' (and 'Save as MAT' for MS-FINDER), setting MS1/MS2 noise thresholds to 2,000 and 100 for a maXis II QTOF; for features with multiple MS/MS spectra, export the one with the most intense precursor ion. Upload the mzML files to GNPS and run the Dereprecation job to match MS/MS spectra against public spectral libraries, then filter the output keeping only hits with the correct polarity mode and a precursor delta m/z within 20 p.p.m. (QTOF). Build the annotation source (filtered GNPS .tsv or a manual CSV with m/z, RT in seconds, Compound name; use -1 in the RT column when RT is unknown) and load it into Optimus's 'Read mz-RT list' node, attaching hits to detected features via 'Match features by mz-RT'. This yields level-2 putative annotations per the Metabolomics Standards Initiative; MS/MS validation of the putative annotations is still required and is not provided in Optimus.

## Related tools

- **Optimus** (KNIME workflow that detects/aligns LC-MS features with OpenMS, exports feature spectra as MGF/MAT, and attaches GNPS-derived annotations via 'Read mz-RT list' and 'Match features by mz-RT') — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the LC-MS feature detection and quantification algorithms used inside Optimus to pair features with their MS/MS spectra) — https://github.com/MolecularCartography/Optimus
- **KNIME** (Workflow management platform hosting the Optimus pipeline including the 'Prepare results', 'Read mz-RT list' and 'Match features by mz-RT' nodes) — https://github.com/MolecularCartography/Optimus
- **GNPS** (Receives uploaded mzML files and runs Dereplication MS/MS matching against public spectral libraries, producing the annotation source exported as .tsv)
- **MS-FINDER** (Alternative downstream tool consuming MAT-format spectral exports of detected features)
- **Sirius** (Alternative downstream tool for In-Silico prediction consuming Optimus's spectral exports)

## Evaluation signals

- annotated_features.csv exists, is a valid CSV, and is the feature quantification matrix with an added compound-annotation field for matched features
- Every GNPS-derived annotation respects the 20 p.p.m. precursor delta m/z tolerance filter and the correct polarity mode
- MGF export contains, for features with several MS/MS spectra, the spectrum with the most intense precursor ion
- MS1/MS2 noise thresholds in 'Prepare results' are 2,000 and 100 respectively before export
- Annotation correctness of individual hits is subject to expert review (level-2 putative annotations only)

## Limitations

- This implements molecular identification at the level 'putatively annotated compounds' (Metabolomics Standards Initiative level 2); MS/MS validation of putative annotations is needed and is currently not provided in Optimus
- Precursor tolerance of 20 p.p.m. applies to QTOF data; other instruments may need different filtering
- RT in the annotation CSV must be given in seconds, and -1 must be used when the retention time is unknown

## Evidence

- [full_text] task_006 finding: [evidence span withheld — anchor status: not_found]
- [methods] task_006 workflow: [evidence span withheld — anchor status: not_found]
