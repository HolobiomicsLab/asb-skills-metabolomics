---
name: charge-state-enumeration-peak-matching
description: Use when a monoisotopic theoretical glycan mass must be matched to an
  observed MS1 peak in an mzXML extract-ion chromatogram across several charge states
  within a ppm tolerance.
when_to_use_negative:
- Sample was acquired by MALDI-TOF or another non-LC platform where charge-state-dependent
  extracted-ion chromatogram peak matching does not apply in the same way
- The glycan of interest is absent from the MS1 Database — the app performs no de
  novo identification
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0081
tools:
- name: GlyKAn AZ
  role: MATLAB app whose MS1 tab executes the per-glycan, per-charge-state peak search
    against the MS1 Database within the ppm tolerance
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ MS1 search algorithm is implemented
    and executed
- name: ProteoWizard
  role: Converts vendor raw MS files into the mzXML format consumed by the MS1 search
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:f11d9b37256514380c7333813461768c4cae84b8fb6d985713405463d3ca3093
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
  tools:
  - GlyKAn AZ
  - MATLAB
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Charge-state enumeration peak matching

## Summary

Enumerates allowed glycan charge states and scans MS1 spectra (mzXML) for peaks matching each glycan's theoretical mass within a user mass-accuracy tolerance (e.g. 10 ppm), assigning the highest-intensity matching peak as the representative apex before isomer detection.

## When to use

Use when a monoisotopic theoretical glycan mass must be matched to an observed MS1 peak in an mzXML extract-ion chromatogram across several charge states within a ppm tolerance.

## When NOT to use

- Sample was acquired by MALDI-TOF or another non-LC platform where charge-state-dependent extracted-ion chromatogram peak matching does not apply in the same way
- The glycan of interest is absent from the MS1 Database — the app performs no de novo identification

## Inputs

- mzXML file from ProteoWizard conversion of Orbitrap LC-MS raw data
- MS1 Database (Excel/tabular) with glycan names, theoretical masses in Da, optional expected RTs in minutes
- User-specified mass-accuracy tolerance (ppm) and charge-state list

## Outputs

- MS1 output Excel file listing glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity per matched peak
- Representative highest-intensity apex (RT, intensity) per glycan/charge state for downstream isomer and MS2 processing

## How to apply

Load the mzXML file plus the MS1 Database (glycan names, theoretical neutral masses in Da, optional expected RTs in minutes) into the MS1 search routine of the GlyKAn AZ app. For each glycan in the database, loop over the allowed charge states and search the MS1 spectra for a peak whose observed m/z equals the theoretical mass divided by that charge (plus proton mass) within the user-specified mass-accuracy tolerance; 10 ppm was the optimal setting for the validated monoclonal-antibody dataset, where 50 ppm introduced 8 extra false positives at the 25% relative-intensity threshold. Select the highest-intensity matching peak as the representative chromatographic apex for that glycan/charge combination. Record the glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity for the match so downstream isomer detection and MS2 confirmation can operate on it.

## Related tools

- **GlyKAn AZ** (MATLAB app whose MS1 tab executes the per-glycan, per-charge-state peak search against the MS1 Database within the ppm tolerance) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment in which the GlyKAn AZ MS1 search algorithm is implemented and executed)
- **ProteoWizard** (Converts vendor raw MS files into the mzXML format consumed by the MS1 search)

## Examples

```
GlyKAn AZ MS1 tab: load sample.mzXML + MS1_Database.xlsx; set mass accuracy = 10 ppm, charge states = {2,3,4}; run search
```

## Evaluation signals

- All 20 manually confirmed glycans are recovered at the same RTs as manual analysis (20 true positives, 0 false positives at 25% relative intensity / 10 ppm)
- Reported mass error per match falls within the configured tolerance (e.g. ≤ 10 ppm)
- Charge state assigned for each match is consistent with the database entry and the observed m/z = theoretical mass relationship
- Every matched peak's intensity is the maximum among all peaks satisfying the mass tolerance for that glycan/charge combination

## Limitations

- Mass-accuracy threshold interacts with relative-intensity threshold: loosening to 50 ppm at 25% relative intensity produced 8 false positives, so tolerance must be tightened when the glycan set is dense
- No de novo identification; only glycans present in the MS1 Database can be matched
- App processes a single mzXML file per run — batch mode is a planned future improvement

## Evidence

- [intro] mzXML and MS1 Database inputs drive per-glycan, per-charge peak search: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
- [results] 10 ppm gave optimal result with all 20 glycans recovered: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [methods] Isomer peaks at same m/z within ±1.75 min use slope-based detection: [evidence span withheld — anchor status: not_found]
- [discussion] App scope limited to N-linked glycans in database: "In addition, the scope of the app could be expanded to consider O-linked glycans, which would enhance its relevance to different fields of research."
- [discussion] App is MATLAB-based with user-friendly interface: "The app itself is MATLAB based and has a user-friendly interface."
- [readme] App streamlines N-linked glycan identification from LC-MS/MS data: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
