---
name: acylcarnitine-flagging-without-msms
description: Use when a urine or other LC-HRMS feature table contains peaks with no
  MS/MS spectra (or MS/MS cannot be used), but accurate m/z and RT were measured on
  HILIC-Q Exactive HF or CSH-Q Exactive HF runs with matched-condition in-house m/z+RT
  libraries available.
when_to_use_negative:
- The feature has an MS/MS spectrum available and library MS/MS matching (e.g. CarniBlast
  acylcarnitine library search) can be performed — use spectral matching for a stronger
  MSI level 2 annotation instead
- The in-house library was acquired under different chromatographic or mass-spectrometric
  conditions than the sample runs, so retention-time windows are not comparable
- The goal is MSI level 1 identification or de novo annotation of compounds absent
  from the in-house library (use authentic standards or in silico tools like SIRIUS/CSI:FingerID
  instead)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: HILIC-Q Exactive HF mass spectrometry
  role: Source of accurate m/z and RT feature measurements under hydrophilic interaction
    chromatography, matched to the correspondingly acquired in-house m/z+RT library
- name: CSH-Q Exactive HF mass spectrometry
  role: Source of accurate m/z and RT feature measurements under charged surface hybrid
    chromatography, matched to the correspondingly acquired in-house m/z+RT library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:24e9ad6c3636bfeb37b6a430a6d579953d7d06a0fefa388308dacb1c0b74cd3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - HILIC-Q Exactive HF mass spectrometry
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# acylcarnitine-flagging-without-msms

## Summary

Annotates acylcarnitines and additional low-abundance compounds at MSI level 2 by matching accurate precursor m/z and retention time against in-house libraries acquired under identical chromatographic and MS conditions, explicitly excluding any MS/MS spectral matching step.

## When to use

Use when a urine or other LC-HRMS feature table contains peaks with no MS/MS spectra (or MS/MS cannot be used), but accurate m/z and RT were measured on HILIC-Q Exactive HF or CSH-Q Exactive HF runs with matched-condition in-house m/z+RT libraries available.

## When NOT to use

- The feature has an MS/MS spectrum available and library MS/MS matching (e.g. CarniBlast acylcarnitine library search) can be performed — use spectral matching for a stronger MSI level 2 annotation instead
- The in-house library was acquired under different chromatographic or mass-spectrometric conditions than the sample runs, so retention-time windows are not comparable
- The goal is MSI level 1 identification or de novo annotation of compounds absent from the in-house library (use authentic standards or in silico tools like SIRIUS/CSI:FingerID instead)

## Inputs

- Feature/peak table with accurate m/z, retention time, and signal intensities from HILIC-Q Exactive HF and/or CSH-Q Exactive HF runs
- In-house m/z and retention-time annotation library acquired under identical chromatographic and mass-spectrometric conditions for each respective assay

## Outputs

- mz_rt_annotations.csv containing all m/z+RT matches tagged with their MSI-confidence rationale (mass and RT only, no spectral confirmation)
- Flagged list of additional acylcarnitine annotations at MSI level 2 (6 in the reference dataset)
- Flagged list of further low-abundance compound annotations (201 in the reference dataset)

## How to apply

Load the feature/peak table containing accurate m/z, retention time, and signal intensities from the HILIC-Q Exactive HF and CSH-Q Exactive HF untargeted urine metabolomics runs. Import the in-house annotation library acquired under identical chromatographic and mass-spectrometric conditions for the respective assay. Match each measured feature to library entries using ppm-level accurate mass plus retention-time tolerance windows only — no MS/MS spectral matching — and tag every match with its MSI-confidence rationale (mass and RT only). Flag the acylcarnitine entries (in this dataset, six additional acylcarnitines beyond MSI level 2 annotations made via CarniBlast with MS/MS) and the further low-abundance library matches (201 compounds here), writing results to mz_rt_annotations.csv.

## Related tools

- **HILIC-Q Exactive HF mass spectrometry** (Source of accurate m/z and RT feature measurements under hydrophilic interaction chromatography, matched to the correspondingly acquired in-house m/z+RT library)
- **CSH-Q Exactive HF mass spectrometry** (Source of accurate m/z and RT feature measurements under charged surface hybrid chromatography, matched to the correspondingly acquired in-house m/z+RT library)

## Evaluation signals

- Every flagged annotation falls within the assay-specific accurate-mass (ppm-level) and retention-time tolerance windows of its corresponding library entry
- Flag counts equal exactly 6 additional acylcarnitines and 201 further low-abundance compounds for the reference urine dataset
- No annotation in the output relied on MS/MS spectral matching — the MSI-confidence rationale recorded is mass and RT only
- Each output row in mz_rt_annotations.csv carries an explicit MSI level 2 confidence tag with the mass+RT rationale
- Matches are restricted to library entries acquired under the same chromatographic and mass-spectrometric conditions as the sample run (HILIC vs CSH assay-specific libraries)

## Limitations

- MSI level 2 annotations from m/z and RT alone lack spectral confirmation, so they are weaker than MS/MS-supported annotations and should not be reported as level 1
- Requires in-house libraries acquired under identical chromatographic and mass-spectrometric conditions; retention-time matching is invalid across differing methods
- Only compounds present in the in-house m/z+RT libraries can be annotated; low-abundance compounds outside library coverage remain unannotated and may need MSI level 3 in silico annotation (e.g. SIRIUS/CSI:FingerID, NIST hybrid search)

## Evidence

- [abstract] MSI level 2 annotation by precursor accurate mass and MS/MS matching: "An additional 578 compounds were annotated by precursor accurate mass and MS/MS matching alone, MSI level 2, including a novel library specifically geared at acylcarnitines (CarniBlast)"
- [abstract] two untargeted metabolomic assays: "two untargeted metabolomic assays, hydrophilic interaction chromatography (HILIC)-Q Exactive HF mass spectrometry and charged surface hybrid (CSH)-Q Exactive HF mass spectrometry"
- [discussion] structure annotation in untargeted MS/MS metabolomics with MSI confidence levels: "structure annotation in untargeted MS/MS metabolomics reports must be annotated with MSI confidence levels"
