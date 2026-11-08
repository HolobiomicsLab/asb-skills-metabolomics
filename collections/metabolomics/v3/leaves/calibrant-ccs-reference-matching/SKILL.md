---
name: calibrant-ccs-reference-matching
description: Use when detected calibrant features (m/z + arrival time) must be assigned
  known CCS values from a reference table (e.g., Agilent TuneMix-CCS) prior to fitting
  a TWIMS polynomial or linearized power calibration function.
when_to_use_negative:
- When CCS can be computed directly from the fundamental low-field ion mobility equation,
  e.g., stepped-field DTIMS-MS runs where --mode multi is used without a calibrant
  file
- When the input is raw instrument frames (e.g., UIMF) rather than already-extracted
  feature tables, since matching operates on feature CSVs
- When calibrant CCS reference values for the tune-mix composition are unavailable
  or the sample is not a recognized tunemix_sample_type
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AutoCCS
  role: Performs the calibrant-to-reference matching (--ppm tolerance, --tunemix_sample_type
    AgilentTuneMix) and uses the matched pairs to fit the TWIMS/DTIMS calibration
    function
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime environment (pandas) in which the calibrant reference table and feature
    files are loaded and matched
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:5981b5781d6700d04d32f5e5047a9b159acd2e2ec6abb12c69057fbb3906964a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - Python 3.7
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# Calibrant CCS reference matching

## Summary

Matches calibrant ion features (e.g., Agilent tune-mix compounds) against a reference arrival-time/CCS table within a ppm tolerance so the matched pairs can drive TWIMS/DTIMS calibration function fitting in AutoCCS.

## When to use

Use when detected calibrant features (m/z + arrival time) must be assigned known CCS values from a reference table (e.g., Agilent TuneMix-CCS) prior to fitting a TWIMS polynomial or linearized power calibration function.

## When NOT to use

- When CCS can be computed directly from the fundamental low-field ion mobility equation, e.g., stepped-field DTIMS-MS runs where --mode multi is used without a calibrant file
- When the input is raw instrument frames (e.g., UIMF) rather than already-extracted feature tables, since matching operates on feature CSVs
- When calibrant CCS reference values for the tune-mix composition are unavailable or the sample is not a recognized tunemix_sample_type

## Inputs

- Calibrant reference table of known CCS values with arrival times (e.g., TuneMix-CCS.txt, Agilent tune-mix)
- Feature CSV files containing detected calibrant peaks (m/z, arrival time) exported per dataset
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns
- autoCCS_config.xml configuration file

## Outputs

- Matched calibrant arrival-time/CCS pairs used as regression input
- calibration_coefficients.csv with fitted coefficients per ionization polarity and platform
- Log output (e.g., LogFiles/single.log or slim.log)

## How to apply

Load the calibrant reference table (e.g., data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt from the MassIVE MSV000085979 demo) and the corresponding feature CSVs with pandas in the Python 3.7 AutoCCS environment. Flag calibrant runs in the sample metadata (Datasets.csv) via --tunemix_sample_type AgilentTuneMix, with --colname_for_sample_type SampleType, --colname_for_filename RawFileName and --colname_for_ionization IonPolarity so matching is done per polarity. Match each calibrant feature to a reference entry by m/z within the ppm tolerance (--ppm 150 in the SLIM demo) and by arrival time, after applying the platform-specific time adjustment (subtract accumulation/access time tacc for SLIM-based IMS; apply the EDC delay coefficient for Waters Synapt). Feed the matched arrival-time/CCS pairs into the calibration fit (--calib_method poly with --degree 2 or 3, or power per Ruotolo et al. 2008), then verify predicted calibrant CCS values reproduce the reference values within the matching tolerance.

## Related tools

- **AutoCCS** (Performs the calibrant-to-reference matching (--ppm tolerance, --tunemix_sample_type AgilentTuneMix) and uses the matched pairs to fit the TWIMS/DTIMS calibration function) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime environment (pandas) in which the calibrant reference table and feature files are loaded and matched) — https://www.python.org/downloads/

## Examples

```
python autoCCS.py --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --tunemix_sample_type AgilentTuneMix --colname_for_sample_type SampleType --colname_for_ionization IonPolarity --output_dir data/SingleField-RapidFire-DTIMS/Results/ --mode single
```

## Evaluation signals

- calibration_coefficients.csv exists and contains a coefficient field for every fitted parameter (polynomial degree terms, or power-function exponent/prefactor plus time offset)
- Predicted calibrant CCS values from the fitted function agree with the reference tune-mix CCS values within the ppm tolerance used for matching (--ppm 150 in the SLIM demo)
- Calibrant features are matched separately for each ionization polarity, consistent with coefficients being written per polarity
- The number of matched calibrant pairs is sufficient for the chosen --degree (quadratic/cubic) without overfitting

## Limitations

- Matching accuracy depends on the ppm tolerance; the demo uses --ppm 150, which may be too loose or tight for other calibrant mixtures
- Platform-specific arrival-time adjustments (subtracting tacc for SLIM, EDC delay for Waters Synapt) must be applied before matching or the matched times will be biased
- The article's discussion section is absent, so no known limitations or reproducibility gaps could be extracted from the source document

## Evidence

- [readme] Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
- [methods] CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: "CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms"
- [abstract] AutoCCS provides accurate and reproducible CCS determination for standard and unknown analyte ions across IMS-MS platforms: "We demonstrated that AutoCCS offers an accurate and reproducible determination of CCS for both standard and unknown analyte ions in various IMS-MS platforms, IMS-field methods, ionization modes and"
