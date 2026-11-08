---
name: mz-rt-molecular-annotation
description: Use when detected LC-MS features need putative compound identities (MSI
  level 2), e.g. annotating rosmarinic acid, urocanate, or sphingosines before spatial
  mapping in 'ili.
when_to_use_negative:
- When definitive compound identification is required — mz-RT matching only yields
  putative level-2 annotations, and Optimus provides no MS/MS validation of the putative
  annotations.
- When input is already an annotated feature table (e.g. a filtered GNPS .tsv already
  joined to intensities) — re-running the matching branch would be redundant.
- When GNPS hits fall outside the 20 p.p.m. QTOF precursor tolerance or have the wrong
  polarity mode — these must be discarded rather than used as the mz-RT list.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow performing feature detection, quantification, and mz-RT matching
    annotation via 'Read mz-RT list' and 'Match features by mz-RT' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: KNIME
  role: Workflow management platform hosting the Optimus annotation pipeline
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the LC-MS feature detection and alignment algorithms underlying feature-spectrum
    pairing
  repo: http://www.openms.de
- name: GNPS
  role: MS/MS dereplication against public spectral libraries; source of the mz-RT
    molecules-of-interest list
  repo: http://gnps.ucsd.edu/
- name: MS-FINDER
  role: Third-party in-silico tool consuming the MAT spectral export for further annotation
- name: Sirius
  role: Third-party in-silico prediction tool consuming the MGF spectral export
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:4ff317171a5471c4933d8bf37d537e7355615fed16b429d689b8e41cb629a8c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - '''ili'
  - GNPS
  - KNIME
  - MS-FINDER
  - OpenMS
  - Optimus
  - OptimusViewer
  - Python
  - Sirius
  - pyopenms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# mz-rt-molecular-annotation

## Summary

Attaches level-2 putative compound annotations to LC-MS features detected with Optimus by mz-RT matching against a molecules-of-interest list exported from GNPS dereplication or hand-built as a CSV of m/z, RT (s), and compound name.

## When to use

Use when detected LC-MS features need putative compound identities (MSI level 2), e.g. annotating rosmarinic acid, urocanate, or sphingosines before spatial mapping in 'ili.

## When NOT to use

- When definitive compound identification is required — mz-RT matching only yields putative level-2 annotations, and Optimus provides no MS/MS validation of the putative annotations.
- When input is already an annotated feature table (e.g. a filtered GNPS .tsv already joined to intensities) — re-running the matching branch would be redundant.
- When GNPS hits fall outside the 20 p.p.m. QTOF precursor tolerance or have the wrong polarity mode — these must be discarded rather than used as the mz-RT list.

## Inputs

- Centroided mzML LC-MS/MS runs loaded into the Optimus KNIME workflow
- features_quantification_matrix.csv of aligned, detected features
- GNPS dereplication output (.tsv) filtered by polarity mode and 20 p.p.m. precursor tolerance, OR a manually created CSV with columns m/z, RT (s), Compound name
- spectra_ms2.mgf exported by Optimus ('Save as MGF')

## Outputs

- annotated_features.csv — feature quantification matrix with an added compound-annotation field for matched features
- Level-2 putative molecular annotations (Metabolomics Standards Initiative) attached to consensus feature IDs

## How to apply

First detect and align features in the Optimus KNIME workflow, setting 'Half of MS/MS isolation window (Da)' to the precursor isolation half-width so each feature is paired with its MS/MS spectrum. Enable MS/MS export in the 'Prepare results' node ('Save as MGF' for GNPS/Sirius, 'Save as MAT' for MS-FINDER), using MS1/MS2 noise thresholds of 2,000 and 100 for a maXis II QTOF; for features with several MS/MS spectra, the one with the most intense precursor ion is exported. Run a GNPS dereplication job on the mzML files, then filter the output to hits with the correct polarity mode and precursor delta m/z within 20 p.p.m. (QTOF tolerance). Build the annotation source from the filtered GNPS .tsv or a manual CSV with three columns (m/z, RT in seconds, compound name), using -1 in the RT column when the retention time is unknown. Load this file into the 'Read mz-RT list' node and attach hits to detected features via 'Match features by mz-RT'; MS/MS validation of putative annotations is not performed by Optimus itself and requires expert review.

## Related tools

- **Optimus** (KNIME workflow performing feature detection, quantification, and mz-RT matching annotation via 'Read mz-RT list' and 'Match features by mz-RT' nodes) — https://github.com/MolecularCartography/Optimus
- **KNIME** (Workflow management platform hosting the Optimus annotation pipeline) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the LC-MS feature detection and alignment algorithms underlying feature-spectrum pairing) — http://www.openms.de
- **GNPS** (MS/MS dereplication against public spectral libraries; source of the mz-RT molecules-of-interest list) — http://gnps.ucsd.edu/
- **MS-FINDER** (Third-party in-silico tool consuming the MAT spectral export for further annotation)
- **Sirius** (Third-party in-silico prediction tool consuming the MGF spectral export) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
pip install six pandas protobuf pymspec pyopenms
```

## Evaluation signals

- annotated_features.csv exists and is a CSV containing the feature quantification matrix plus an added compound-annotation field for matched features.
- Every GNPS-derived annotation respects the 20 p.p.m. precursor delta m/z filter and the correct polarity mode.
- Annotation list CSV has exactly three columns (m/z, RT in seconds, Compound name); unknown retention times are encoded as -1 in the RT column.
- Known targets are recovered in the validation cases: rosmarinic acid, methoxycarnosic acid, and cirsimaritin in rosemary extracts; urocanate at m/z 139.050 in ATM keypad swabs.
- Residual correctness of individual annotation hits is subject to expert review — flagged hits should be spot-checked against MS/MS spectra in OptimusViewer.

## Limitations

- mz-RT matching implements identification only at the level 'putatively annotated compounds' (MSI level 2); MS/MS validation of putative annotations is needed and is currently not provided in Optimus.
- For features with several MS/MS spectra, only the spectrum with the most intense precursor ion is exported, so lower-quality spectra are not available for validation.
- GNPS dereplication hits must be manually filtered by polarity mode and 20 p.p.m. tolerance before use; unfiltered output produces wrong annotations.
- MS1/MS2 noise thresholds (2,000/100) and the 20 p.p.m. tolerance are instrument-dependent — Orbitrap data would need different settings.

## Evidence

- [methods] Filter the GNPS output keeping only hits with the correct polarity mode and a precursor delta m/z within 20 p.p.m. (QTOF), and download the filtered .tsv.: [evidence span withheld — anchor status: not_found]
- [methods] the output annotated_features.csv must exist, be a CSV containing the feature quantification matrix with an added compound-annotation field for matched features: [evidence span withheld — anchor status: not_found]
