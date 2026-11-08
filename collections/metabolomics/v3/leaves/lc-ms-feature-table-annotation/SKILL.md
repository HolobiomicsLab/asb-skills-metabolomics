---
name: lc-ms-feature-table-annotation
description: Use when you have a feature/peak table with accurate m/z, RT, and intensities
  from a fixed LC-MS assay (e.g., HILIC- or CSH-Q Exactive HF) plus a same-condition
  in-house m/z+RT library, and MS/MS spectra are absent, sparse, or not required for
  level 2 annotation.
when_to_use_negative:
- MS/MS spectra are available and matched — precursor accurate mass plus MS/MS matching
  alone annotates 578 compounds at MSI level 2 in this study, which is the stronger
  evidence route; m/z+RT-only matching is the fallback for features without usable
  MS/MS.
- The in-house library was acquired under different chromatographic or mass-spectrometric
  conditions — RT windows are then meaningless and matches would be spurious.
- You need MSI level 1 identification — this skill's output is level 2 by definition
  and cannot substitute for authentic standards with precursor mass, retention time,
  AND MS/MS matching.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HILIC-Q Exactive HF mass spectrometry
  role: Source instrument for the HILIC urine feature table (accurate m/z, RT, intensities)
    whose same-condition library is used for level 2 m/z+RT annotation
- name: CSH-Q Exactive HF mass spectrometry
  role: Source instrument for the CSH urine feature table whose same-condition m/z+RT
    library is used for level 2 m/z+RT annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:020166e107e3e4255d1ae6908079f36209a7bb7e19cc1bd0fdf3df6da3a7a723
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - HILIC-Q Exactive HF mass spectrometry
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# lc-ms-feature-table-annotation

## Summary

Annotate LC-MS features at MSI level 2 by matching accurate precursor m/z and retention time against in-house libraries acquired under identical chromatographic and MS conditions, explicitly without MS/MS spectral matching. This recovers additional annotations (e.g., six further acylcarnitines and 201 low-abundance compounds in urine) that MS/MS-only workflows miss.

## When to use

Use when you have a feature/peak table with accurate m/z, RT, and intensities from a fixed LC-MS assay (e.g., HILIC- or CSH-Q Exactive HF) plus a same-condition in-house m/z+RT library, and MS/MS spectra are absent, sparse, or not required for level 2 annotation.

## When NOT to use

- MS/MS spectra are available and matched — precursor accurate mass plus MS/MS matching alone annotates 578 compounds at MSI level 2 in this study, which is the stronger evidence route; m/z+RT-only matching is the fallback for features without usable MS/MS.
- The in-house library was acquired under different chromatographic or mass-spectrometric conditions — RT windows are then meaningless and matches would be spurious.
- You need MSI level 1 identification — this skill's output is level 2 by definition and cannot substitute for authentic standards with precursor mass, retention time, AND MS/MS matching.

## Inputs

- Urine feature/peak table with accurate m/z, retention time, and signal intensities from HILIC-Q Exactive HF and CSH-Q Exactive HF untargeted metabolomics runs
- In-house m/z and retention-time annotation libraries acquired under identical chromatographic and mass-spectrometric conditions for each respective assay

## Outputs

- mz_rt_annotations.csv: all m/z+RT-only annotations, each tagged with its MSI-confidence rationale (mass and RT only, no spectral confirmation), including flagged acylcarnitine entries and further low-abundance compound entries

## How to apply

Load the urine feature/peak table containing accurate m/z, retention time, and signal intensities from the HILIC-Q Exactive HF and CSH-Q Exactive HF runs. Import the in-house m/z and RT annotation libraries acquired under identical chromatographic and mass-spectrometric conditions for the respective assay — condition matching is essential because RT is only comparable within the same method. Match each measured feature to library entries using accurate-mass (ppm-level) plus retention-time windows only, explicitly excluding any MS/MS spectral matching step. Flag acylcarnitine and other library-matched entries, and write all flags to mz_rt_annotations.csv tagged with their MSI-confidence rationale (mass and RT only, no spectral confirmation). Validation requires every flag to fall within the assay-specific accurate-mass and RT tolerance windows of its library entry.

## Related tools

- **HILIC-Q Exactive HF mass spectrometry** (Source instrument for the HILIC urine feature table (accurate m/z, RT, intensities) whose same-condition library is used for level 2 m/z+RT annotation)
- **CSH-Q Exactive HF mass spectrometry** (Source instrument for the CSH urine feature table whose same-condition m/z+RT library is used for level 2 m/z+RT annotation)

## Evaluation signals

- Every flagged annotation falls within the assay-specific accurate-mass (ppm-level) and RT tolerance windows of its corresponding library entry.
- Flag counts equal exactly 6 additional acylcarnitines and 201 further low-abundance compounds in the urine dataset.
- No annotation in the output relies on MS/MS spectral matching; each carries the MSI-confidence rationale 'mass and RT only, no spectral confirmation'.
- Each feature-library match is assay-specific (HILIC features matched against the HILIC-condition library, CSH features against the CSH-condition library).

## Limitations

- MSI level 2 confidence from m/z and RT alone is lower than level 2 annotations backed by MS/MS matching; the field lacks generally accepted and validated automated confidence calculations with false-discovery rate assessments in metabolomics MS/MS.
- Only compounds represented in the same-condition in-house libraries can be annotated — coverage is limited to the 6 acylcarnitines and 201 low-abundance compounds the libraries span in this dataset.
- Retention-time matching is only valid when the library was acquired under identical chromatographic and mass-spectrometric conditions.

## Evidence

- [other] Card finding on m/z+RT-only level 2 annotations: [evidence span withheld — anchor status: not_found]
