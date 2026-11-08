---
name: replicate-retention-time-rsd-assessment
description: Use when building or validating a retention-time reference library from
  chemical-standard mixtures measured in replicate LC-MS (e.g. three HILIC injections
  per mixture), and you must decide which analytes' retention times are reproducible
  enough to record.
when_to_use_negative:
- You only need to rank mobile phases, not validate retention times — use the MetaMOPE
  mobile-phase scoring step (peak quality score = Sp + Saf) instead.
- Retention times come from a single injection per standard — with no three replicates,
  retention-time RSD across replicates cannot be computed and thresholds c/r/I cannot
  be jointly applied.
- Input is already a validated feature/retention-time table; this skill generates
  validation flags, it does not re-validate annotated libraries.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMOPE
  role: Containerized web tool that generates the eight per-peak metrics (including
    retention-time RSD and Std/Blk ratio) from replicate EICs and applies the user
    thresholds to produce the validation column
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment runtime for MetaMOPE (docker-compose up --build) providing the
    workflow locally
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Source code of functions for the peak metrics written in R, used within the
    MetaMOPE workflow
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:e0523ecfb1fca7d4cf94a33f16bc9681f3d18c4322f7b0262b4e211107abfbe9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# replicate-retention-time-rsd-assessment

## Summary

Assesses reproducibility of analyte retention times across replicate LC-MS injections by computing the retention-time RSD per standard peak and applying user-specified thresholds (RSD < c, Std/Blk ratio > r, intensity > I) to validate metabolites before building a retention-time reference library.

## When to use

Use when building or validating a retention-time reference library from chemical-standard mixtures measured in replicate LC-MS (e.g. three HILIC injections per mixture), and you must decide which analytes' retention times are reproducible enough to record.

## When NOT to use

- You only need to rank mobile phases, not validate retention times — use the MetaMOPE mobile-phase scoring step (peak quality score = Sp + Saf) instead.
- Retention times come from a single injection per standard — with no three replicates, retention-time RSD across replicates cannot be computed and thresholds c/r/I cannot be jointly applied.
- Input is already a validated feature/retention-time table; this skill generates validation flags, it does not re-validate annotated libraries.

## Inputs

- Per-peak metrics for three replicate injections per standard mixture (retention-time RSD, Std/Blk intensity ratio, maximum intensity, plus MCQ, jaggedness, asymmetry factor, FWHM, modality)
- Standard analytes' extracted ion chromatograms from mzXML/mzML files
- User-specified thresholds r (Std/Blk ratio), c (retention-time RSD), and I (peak intensity)

## Outputs

- Validation flag per metabolite indicating whether thresholds are met
- Retention-time reference library table (validation column plus validated retention time per metabolite, e.g. 571 metabolites for the HILIC LC-MS platform)

## How to apply

For each standard mixture injected three times, retrieve the extracted ion chromatograms (from mzXML/mzML inputs) and take the retention time determined from the EICs of the three replicates after IRLS baseline correction and Savitzky-Golay smoothing. Compute the RSD of the retention time across replicates, together with the Std/Blk intensity ratio and peak intensity. Record a peak only if its Std/Blk ratio exceeds the user-specified threshold r, its retention-time RSD falls below the user-specified threshold c, and its intensity exceeds the user-specified threshold I. Mark each metabolite as validated or not in the validation column of the output table, so only validated metabolites carry a retention-time entry in the final reference library (e.g. the 571-metabolite HILIC LC-MS library).

## Related tools

- **MetaMOPE** (Containerized web tool that generates the eight per-peak metrics (including retention-time RSD and Std/Blk ratio) from replicate EICs and applies the user thresholds to produce the validation column) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment runtime for MetaMOPE (docker-compose up --build) providing the workflow locally) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Source code of functions for the peak metrics written in R, used within the MetaMOPE workflow) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build  # then access http://localhost:13002 to run MetaMOPE with thresholds r, c, I
```

## Evaluation signals

- Every recorded peak satisfies all three criteria: Std/Blk > r, retention-time RSD < c across the three replicates, and intensity > I.
- The output table contains a validation column whose flags agree with threshold satisfaction for each metabolite.
- Only validated metabolites carry an extracted retention time taken from the EICs of the three replicates.
- Library size and format match the reference outcome (e.g. 571 validated metabolites for the HILIC LC-MS platform).

## Limitations

- Thresholds r, c, and I are user-specified, so validation outcomes depend on chosen cutoffs; no default values are documented in the provided context.
- Peaks failing any criterion are simply not recorded — the discussion of limitations could not be extracted because the document stated 'No changelog found.'
- Workflow assumes each standard mixture was evaluated exactly three times by the LC/MS method.

## Evidence

- [abstract] Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation"
- [abstract] MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [abstract] The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates.: "The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates."
- [abstract] The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds.: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [results] This workflow was successfully applied to construct a reference library of 571 metabolites: "This workflow was successfully applied to construct a reference library of 571 metabolites"
