---
name: low-abundance-compound-annotation
description: Use when the urine feature table contains low-abundance peaks lacking
  MS/MS fragmentation but with accurate mass and retention time that fall within condition-matched
  in-house library tolerance windows for m/z and RT.
when_to_use_negative:
- When the feature has an MS/MS spectrum, use precursor accurate mass plus MS/MS matching
  (MSI level 2 with spectral evidence) or CSI:FingerID/NIST hybrid search for MSI
  level 3 instead of m/z+RT-only annotation.
- When the in-house m/z/RT library was acquired on different chromatography or instrument
  conditions, RT-matching windows are not valid and this skill should not be applied.
- When the goal is highest-confidence MSI level 1 identification, authentic standards
  with precursor mass, RT, and MS/MS matching are required; m/z+RT matching alone
  cannot reach level 1.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: HILIC-Q Exactive HF mass spectrometry
  role: Generates accurate-mass and retention-time feature data for low-abundance
    urine compounds under hydrophilic interaction chromatography
- name: CSH-Q Exactive HF mass spectrometry
  role: Second untargeted assay providing condition-matched m/z and RT data for low-abundance
    compound annotation
- name: CarniBlast
  role: Novel acylcarnitine-oriented library supplying m/z and RT entries for the
    six additional acylcarnitine annotations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:18714df1cf408c55c51369b878efb588985d68f60c4e6e7e129e3fd03bdda220
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - CarniBlast
  - HILIC-Q Exactive HF mass spectrometry
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# low-abundance-compound-annotation

## Summary

Annotates low-abundance metabolite signals in untargeted LC-HRMS urine data by matching accurate precursor m/z and retention time against in-house m/z/RT libraries acquired under identical chromatographic and instrument conditions (HILIC-Q Exactive HF and CSH-Q Exactive HF), without requiring MS/MS spectra.

## When to use

Use when the urine feature table contains low-abundance peaks lacking MS/MS fragmentation but with accurate mass and retention time that fall within condition-matched in-house library tolerance windows for m/z and RT.

## When NOT to use

- When the feature has an MS/MS spectrum, use precursor accurate mass plus MS/MS matching (MSI level 2 with spectral evidence) or CSI:FingerID/NIST hybrid search for MSI level 3 instead of m/z+RT-only annotation.
- When the in-house m/z/RT library was acquired on different chromatography or instrument conditions, RT-matching windows are not valid and this skill should not be applied.
- When the goal is highest-confidence MSI level 1 identification, authentic standards with precursor mass, RT, and MS/MS matching are required; m/z+RT matching alone cannot reach level 1.

## Inputs

- Feature/peak table from untargeted LC-HRMS urine runs containing accurate m/z, retention time, and signal intensities (HILIC-Q Exactive HF and/or CSH-Q Exactive HF)
- In-house m/z and RT annotation libraries acquired under identical chromatographic and mass-spectrometric conditions (including CarniBlast acylcarnitine entries)

## Outputs

- mz_rt_annotations.csv listing low-abundance compounds annotated at MSI level 2 with m/z, RT, and MSI-confidence rationale (mass and RT only, no spectral confirmation)
- Flagged acylcarnitine annotations from CarniBlast matched by m/z and RT

## How to apply

Load the measured feature table from the HILIC-Q Exactive HF or CSH-Q Exactive HF untargeted urine assay, then import the assay-specific in-house m/z and retention-time library that was acquired under the same chromatographic and mass-spectrometric conditions. Match each feature using ppm-level accurate-mass windows plus RT windows only, explicitly excluding MS/MS spectral matching. Flag matches as MSI level 2 annotations and record the confidence rationale (mass and RT only, no spectral confirmation). In this study this yielded six additional acylcarnitines and 201 further low-abundance compounds beyond the 578 MS/MS-based MSI level 2 annotations.

## Related tools

- **HILIC-Q Exactive HF mass spectrometry** (Generates accurate-mass and retention-time feature data for low-abundance urine compounds under hydrophilic interaction chromatography)
- **CSH-Q Exactive HF mass spectrometry** (Second untargeted assay providing condition-matched m/z and RT data for low-abundance compound annotation)
- **CarniBlast** (Novel acylcarnitine-oriented library supplying m/z and RT entries for the six additional acylcarnitine annotations)

## Examples

```
import pandas as pd; feats = pd.read_csv('urine_features.csv'); lib = pd.read_csv('hilic_mzrt_library.csv'); matches = feats.merge(lib, on=None, suffixes=('_meas','_lib'))
```

## Evaluation signals

- Every flagged annotation must fall within the assay-specific accurate-mass (ppm-level) and RT tolerance windows of its corresponding library entry.
- Flag counts must equal exactly 6 acylcarnitines and 201 further low-abundance compounds for this urine dataset.
- No flagged annotation should carry MS/MS spectral evidence; all rationale must state mass and RT only.
- All annotations must be tagged MSI level 2, not level 1 or 3.
- Library and sample runs must be verifiably acquired under the same chromatographic and instrument conditions per assay.

## Limitations

- Annotations rely solely on accurate mass and retention time without spectral confirmation, so isomeric compounds sharing m/z and RT windows cannot be distinguished.
- The approach requires in-house libraries acquired under identical chromatographic and mass-spectrometric conditions; it does not transfer to data from other conditions.
- The field lacks generally accepted, validated automated calculations of identification confidence levels with false-discovery rate assessments in metabolomics MS/MS, so MSI level 2 assignments from m/z+RT alone should be reported with explicit confidence rationale.

## Evidence

- [methods] six further acylcarnitines annotated at MSI level 2 using only m/z and RT: [evidence span withheld — anchor status: not_found]
