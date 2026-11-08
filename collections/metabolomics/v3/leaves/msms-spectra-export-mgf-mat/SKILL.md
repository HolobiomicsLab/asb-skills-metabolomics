---
name: msms-spectra-export-mgf-mat
description: Use when detected LC-MS features have associated MS/MS spectra and you
  need portable spectral files (MGF/MAT) for GNPS spectral-library dereplication,
  MS-FINDER, or Sirius annotation.
when_to_use_negative:
- Features lack acquired MS/MS spectra — apply the optional 'Exclusion of features
  for which MS/MS spectra were not acquired' filter first, since only features with
  MS/MS can be exported for library matching
- You already have a GNPS dereplication result — proceed directly to building the
  mz-RT annotation list rather than re-exporting spectra
- The goal is only intensity quantification/normalization or spatial mapping in 'ili,
  which does not require spectral export
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow whose 'Prepare results' node performs the MGF/MAT spectral
    export of detected features
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the LC-MS feature detection/quantification algorithms that pair features
    with their MS/MS spectra before export
- name: KNIME
  role: Workflow management platform hosting the Optimus export nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: GNPS
  role: Downstream consumer of the exported MGF for MS/MS matching against public
    spectral libraries
- name: MS-FINDER
  role: Downstream in-silico annotation tool consuming the exported MAT file
- name: Sirius
  role: Downstream in-silico prediction tool consuming the exported MGF
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:3bfc1d20c4aee9511b3316bd084e37554785c83fd68cd35052af5ee4a52e5612
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
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-spectra-export-mgf-mat

## Summary

Exports MS/MS spectra of LC-MS features detected with the Optimus KNIME workflow into MGF (for GNPS dereplication or Sirius) and MAT (for MS-FINDER) formats so spectra can be matched against public spectral libraries and used for downstream in-silico annotation.

## When to use

Use when detected LC-MS features have associated MS/MS spectra and you need portable spectral files (MGF/MAT) for GNPS spectral-library dereplication, MS-FINDER, or Sirius annotation.

## When NOT to use

- Features lack acquired MS/MS spectra — apply the optional 'Exclusion of features for which MS/MS spectra were not acquired' filter first, since only features with MS/MS can be exported for library matching
- You already have a GNPS dereplication result — proceed directly to building the mz-RT annotation list rather than re-exporting spectra
- The goal is only intensity quantification/normalization or spatial mapping in 'ili, which does not require spectral export

## Inputs

- mzML LC-MS/MS runs processed through Optimus feature detection/alignment
- MS/MS spectra paired to detected features via the MS/MS isolation-window setting
- MS1/MS2 noise thresholds (e.g. 2,000 and 100 for a maXis II QTOF)

## Outputs

- MGF file of feature MS/MS spectra for GNPS dereplication or Sirius
- MAT file of feature MS/MS spectra for MS-FINDER
- Feature quantification matrix (.db) consumable by OptimusViewer

## How to apply

First run Optimus feature detection with OpenMS-based nodes, setting 'Half of MS/MS isolation window (Da)' to the half-width of the MS/MS precursor isolation window so each feature is paired with its correct MS/MS spectrum. Then enable the MS/MS export in the 'Prepare results' node by checking 'Save as MGF' (and 'Save as MAT' when MS-FINDER is the downstream tool), setting MS1/MS2 noise thresholds to 2,000 and 100 respectively for a maXis II QTOF instrument. When a feature has several acquired MS/MS spectra, Optimus exports only the spectrum with the most intense precursor ion. The resulting MGF is uploaded with the mzML files to GNPS for MS/MS matching against public spectral libraries, and the MAT file feeds MS-FINDER.

## Related tools

- **Optimus** (KNIME workflow whose 'Prepare results' node performs the MGF/MAT spectral export of detected features) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the LC-MS feature detection/quantification algorithms that pair features with their MS/MS spectra before export)
- **KNIME** (Workflow management platform hosting the Optimus export nodes) — https://github.com/MolecularCartography/Optimus
- **GNPS** (Downstream consumer of the exported MGF for MS/MS matching against public spectral libraries)
- **MS-FINDER** (Downstream in-silico annotation tool consuming the exported MAT file)
- **Sirius** (Downstream in-silico prediction tool consuming the exported MGF)

## Evaluation signals

- The exported MGF contains one MS/MS spectrum per feature, and for features with multiple spectra the exported spectrum has the most intense precursor ion
- The precursor isolation window was set correctly ('Half of MS/MS isolation window (Da)'), so spectra in the MGF correspond to the right precursor m/z values
- GNPS dereplication of the uploaded MGF returns hits whose precursor delta m/z falls within 20 p.p.m. (QTOF) after polarity filtering
- The MAT file loads in MS-FINDER and the MGF loads in Sirius without format errors
- Feature IDs in the exported spectral files map 1:1 to rows of the Optimus feature quantification matrix

## Limitations

- Only one MS/MS spectrum per feature is exported (the one with the most intense precursor ion), so lower-quality co-acquired spectra are lost
- Export is gated on MS/MS having been acquired for a feature; features without MS/MS spectra cannot be dereplicated this way
- Noise thresholds (MS1 2,000 / MS2 100) are instrument-specific (maXis II QTOF in the protocol) and may need retuning on other instruments
- GNPS-derived annotations remain MSI level-2 putative annotations; MS/MS validation of individual hits is not provided within Optimus and requires expert review

## Evidence

- [methods] Spectral export format and instrument thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Multiple-spectrum handling rule: [evidence span withheld — anchor status: not_found]
- [readme] OptimusViewer db output contains annotations: [evidence span withheld — anchor status: external_doc]
