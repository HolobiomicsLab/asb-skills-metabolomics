---
name: saturation-repair-peak-reconstruction
description: Use when LC-IM-MS raw/profile-level data (e.g. Agilent .d TOF files)
  contain high-abundance ions with truncated intensity profiles due to detector saturation,
  and accurate isotopic distributions or monoisotopic peak heights are needed downstream.
when_to_use_negative:
- Data are already centroided or processed to a peak/feature table, since the algorithm
  operates on raw/profile-level spectra and needs the full intensity profile to reconstruct
  saturated peaks
- Input is low-abundance data with no ions near detector saturation — the smoothing
  step (not saturation repair) is the module that addresses poorly defined low-abundance
  signals
- A theoretical isotopic distribution is already available and the user only wants
  to fit a model rather than recover the observed undistorted distribution from the
  same run
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software housing the saturation repair algorithm; generates a new
    MS-file in the same instrument format with enhanced signal quality.
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: ChemCalc
  role: Used to compute theoretical isotopic distributions against which the repaired
    isotopic distribution's Pearson correlation score is judged.
- name: R
  role: Environment in which isotopic Pearson-correlation scores (repaired distribution
    vs. ChemCalc theoretical distributions) are computed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:fbab425f1b155e8a5b3c857aefbfff0957ac21dede1394936321941bc3e24d25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemCalc
  - PNNL PreProcessor
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# saturation-repair-peak-reconstruction

## Summary

The PNNL PreProcessor saturation repair module reconstructs the distorted LC and IM intensity profiles of high-abundance ions whose peaks are truncated by detector saturation in LC-IM-MS data, recovering the undistorted isotopic distribution from later, unsaturated portions of the run.

## When to use

Use when LC-IM-MS raw/profile-level data (e.g. Agilent .d TOF files) contain high-abundance ions with truncated intensity profiles due to detector saturation, and accurate isotopic distributions or monoisotopic peak heights are needed downstream.

## When NOT to use

- Data are already centroided or processed to a peak/feature table, since the algorithm operates on raw/profile-level spectra and needs the full intensity profile to reconstruct saturated peaks
- Input is low-abundance data with no ions near detector saturation — the smoothing step (not saturation repair) is the module that addresses poorly defined low-abundance signals
- A theoretical isotopic distribution is already available and the user only wants to fit a model rather than recover the observed undistorted distribution from the same run

## Inputs

- Filtered MS-file from the prior PreProcessor step (raw/profile-level LC-IM-MS data, e.g. Agilent .d or UIMF)
- User-defined abundance-limit percentage threshold (e.g. 40%)

## Outputs

- Repaired MS-file in the same instrument format, profile mode, with '_SR' suffix in the file name
- Reconstructed LC and IM intensity profiles of saturated isotopic peaks
- Recovered undistorted isotopic distributions (e.g. tripling the monoisotopic peak height, reducing mass error from 10.6 to 1.9 pm)

## How to apply

Run the saturation repair module of the PNNL PreProcessor on the filtered MS-file from the prior preprocessing step, keeping output in the same instrument format in profile mode and appending a '_SR' suffix to the file name. Set the saturation threshold to 40% of the abundance limit (as done for the human plasma lipid-extract LC-DT IM-MS triplicates). The algorithm automatically finds ions exceeding this user-defined threshold, recovers the undistorted reference isotopic distribution from spectra at later LC or IM times where the ion is unsaturated, and uses the most intense unsaturated isotopic peak together with the reference isotopic ratios to successively reconstruct the LC and IM profiles of the saturated isotopic peaks. Because the method works on raw/profile-level spectra and requires no theoretical isotopic distributions, it is applicable to peptides, metabolites, lipids, glycans, and inorganic polymers. Verify the repaired output file exists, is in the expected instrument format, and structurally matches the deposited preprocessed reference file (MSV000087295) within format/tolerance.

## Related tools

- **PNNL PreProcessor** (Standalone software housing the saturation repair algorithm; generates a new MS-file in the same instrument format with enhanced signal quality.) — https://omics.pnl.gov/software/pnnl-preprocessor
- **ChemCalc** (Used to compute theoretical isotopic distributions against which the repaired isotopic distribution's Pearson correlation score is judged.)
- **R** (Environment in which isotopic Pearson-correlation scores (repaired distribution vs. ChemCalc theoretical distributions) are computed.)

## Examples

```
PNNL-PreProcessor.exe --input plasma_lipid.d --filter filtered.d --saturation-repair 40% --output-mode profile --suffix _SR
```

## Evaluation signals

- Output file exists, is in the expected instrument format (e.g. Agilent .d / UIMF), and is in profile mode with the '_SR' suffix
- Output structurally matches the deposited preprocessed reference file from MassIVE deposit MSV000087295 within format/tolerance
- Isotopic Pearson correlation of the repaired distribution against ChemCalc theoretical distributions (computed in R) is high — targeted recovery tripled the monoisotopic peak height and reduced mass error from 10.6 to 1.9 pm
- Saturated ions are correctly identified as those exceeding 40% of the abundance limit

## Limitations

- Only effective for ions whose signals are truncated due to detector saturation; it does not address poor peak definition of low-abundance ions
- Requires raw/profile-level spectra — not applicable to already centroided or peak-list data
- Applied threshold must be set as a percentage of the abundance limit; incorrect threshold may miss saturated ions or flag unsaturated ones
- No explicit version or quantitative performance metrics for saturation repair alone are given in the discussion text

## Evidence
