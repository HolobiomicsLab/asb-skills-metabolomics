---
name: lcms-standard-blank-intensity-ratio-filtering
description: Use when building or validating a retention-time reference library from
  chemical standard mixtures measured in triplicate LC-MS runs and peaks must pass
  blank-vs-standard (Std/Blk), RSD, and intensity thresholds before recording.
when_to_use_negative:
- Choosing the optimal mobile phase from candidate phases — that uses the peak quality
  score (Sp + Saf) comparison, not the Std/Blk validation filter
- Input is raw chromatograms that have not yet been processed by MetaMOPE into the
  eight per-peak metrics (EIC extraction, IRLS baseline correction, Savitzky-Golay
  smoothing must happen first)
- Peak detection or retention-time determination itself — those are computed from
  MCQ, asymmetry factor, and local maximum intensity; this skill only filters and
  validates already-detected peaks
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMOPE
  role: Containerized web tool that generates the eight per-peak metrics and applies
    the Std/Blk, RSD, and intensity filter criteria to validate metabolites and record
    their retention times
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source code of functions for the multiple peak-evaluation metrics
    used in the filtering workflow
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment vehicle; run the MetaMOPE web service locally via docker-compose
    to execute the filter workflow
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:3391d49f38ca7653aab5618a8fe19ab865479fbc73bd46b96b08ce69f0432894
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lcms-standard-blank-intensity-ratio-filtering

## Summary

Filter candidate metabolite peaks from chemical-standard LC-MS replicate injections using MetaMOPE's Std/Blk intensity-ratio, retention-time RSD, and peak intensity criteria, so that only validated peaks contribute retention times to a reference library.

## When to use

Use when building or validating a retention-time reference library from chemical standard mixtures measured in triplicate LC-MS runs and peaks must pass blank-vs-standard (Std/Blk), RSD, and intensity thresholds before recording.

## When NOT to use

- Choosing the optimal mobile phase from candidate phases — that uses the peak quality score (Sp + Saf) comparison, not the Std/Blk validation filter
- Input is raw chromatograms that have not yet been processed by MetaMOPE into the eight per-peak metrics (EIC extraction, IRLS baseline correction, Savitzky-Golay smoothing must happen first)
- Peak detection or retention-time determination itself — those are computed from MCQ, asymmetry factor, and local maximum intensity; this skill only filters and validates already-detected peaks

## Inputs

- Per-peak metric table from MetaMOPE for three replicate injections of each chemical standard mixture (MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, modality)
- User-specified thresholds r (Std/Blk), c (retention-time RSD), and I (peak intensity)
- Extracted ion chromatograms (EICs) from mzXML/mzML input files of the standard mixture replicates

## Outputs

- Validation flag per metabolite indicating pass/fail against the user-specified thresholds
- Retention-time reference library table (validation flag plus validated retention time per metabolite), matching the 571-metabolite HILIC LC-MS library format

## How to apply

Load the eight per-peak metrics MetaMOPE generated for the three replicate injections of each standard mixture: MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, and modality. Record only peaks satisfying all three user-specified criteria simultaneously: Std/Blk intensity ratio greater than threshold r, retention-time RSD across the three replicates below threshold c, and peak intensity larger than threshold I. Write the validation decision into the 'validation' column, which decides whether metabolites in the sample mixture are validated by the user-specified thresholds. For validated metabolites, extract the retention time determined from the EICs of the three replicates and emit the retention-time reference library (validation flag plus validated retention time per metabolite), in the format of the 571-metabolite reference library reported for the HILIC LC-MS platform. Threshold choices are delegated to the user, so document the r, c, and I values used for reproducibility.

## Related tools

- **MetaMOPE** (Containerized web tool that generates the eight per-peak metrics and applies the Std/Blk, RSD, and intensity filter criteria to validate metabolites and record their retention times) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source code of functions for the multiple peak-evaluation metrics used in the filtering workflow) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment vehicle; run the MetaMOPE web service locally via docker-compose to execute the filter workflow) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
docker-compose up --build && browse to http://localhost:13002 to run MetaMOPE's standard-validation workflow
```

## Evaluation signals

- Every recorded peak satisfies all three criteria simultaneously: Std/Blk > r, retention-time RSD < c across the three replicates, and intensity > I
- The output table has a populated 'validation' column marking each metabolite as validated or not per the thresholds
- Validated metabolites carry a retention time derived from the EICs of all three replicate injections
- Library row count/schema matches the reported reference-library format (e.g. 571 metabolites for the HILIC LC-MS platform when using equivalent standards)
- The r, c, and I threshold values used are documented and reproducible from run parameters

## Limitations

- Thresholds r, c, and I are user-specified, so filter outcomes are not fixed by the method and must be reported per study
- No changelog/discussion content was available in the article, so limitations, missing artifacts, or reproducibility gaps could not be extracted from that section
- Peaks failing any single criterion are excluded entirely — the filter is a joint (AND) condition, so a borderline Rt-RSD or intensity failure discards the peak regardless of Std/Blk

## Evidence

- [abstract] Filter step (abstract): "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
- [abstract] Full criteria triple: "the relative standard deviation (RSD) of the retention time of the detected peaks of the three replicates is less than a user-specified threshold c, and the peak intensity is larger than a"
- [abstract] Validation column semantics: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [abstract] Retention time source: "The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates."
- [results] Library scale/format: "This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform"
- [readme] Deployment invocation: "Run `docker-compose up --build`"
