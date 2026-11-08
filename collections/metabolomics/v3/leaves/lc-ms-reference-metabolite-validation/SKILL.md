---
name: lc-ms-reference-metabolite-validation
description: Use when a raw mzXML/mzML standard-mixture file and its extracted library
  table must be screened so that only metabolite peaks passing Std/Blk ratio, RT RSD,
  and intensity thresholds enter the validated reference library.
when_to_use_negative:
- The goal is mobile-phase optimization (peak quality scoring across candidate mobile
  phases) rather than validating metabolites for the reference library
- Input data is not replicate LC-MS standard-mixture chromatograms (mzXML/mzML) with
  accompanying blanks — the Std/Blk and RT RSD criteria cannot be computed
- The library table already carries a validation column with confirmed entries and
  only needs counting, not threshold-based re-validation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MetaMOPE
  role: Computes the eight peak metrics (MCQ, maximum intensity, RT RSD, Std/Blk,
    jaggedness, asymmetry factor, FWHM, modality) from EICs and sets the validation
    column via user-specified thresholds
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Containerizes and deploys the MetaMOPE web tool used to run the validation
    workflow
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source code of the multiple validation metrics used by MetaMOPE
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:d12dbc45d1ead65b22bc868db856e7352078293164b0c859ad49adba8d322ec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-ms-reference-metabolite-validation

## Summary

Validates metabolites in an LC-MS reference library against user-specified peak-quality thresholds computed by MetaMOPE (MCQ, maximum intensity, retention-time RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality), and verifies the resulting library count (571 metabolites for HILIC LC-MS).

## When to use

Use when a raw mzXML/mzML standard-mixture file and its extracted library table must be screened so that only metabolite peaks passing Std/Blk ratio, RT RSD, and intensity thresholds enter the validated reference library.

## When NOT to use

- The goal is mobile-phase optimization (peak quality scoring across candidate mobile phases) rather than validating metabolites for the reference library
- Input data is not replicate LC-MS standard-mixture chromatograms (mzXML/mzML) with accompanying blanks — the Std/Blk and RT RSD criteria cannot be computed
- The library table already carries a validation column with confirmed entries and only needs counting, not threshold-based re-validation

## Inputs

- Standard mixture mzXML/mzML input files (three replicates per standard mixture)
- Reference library table from the MetaMOPE/MetaHILIC repository with a validation column per metabolite
- User-specified thresholds r (Std/Blk ratio), c (RT RSD), and I (peak intensity)

## Outputs

- Reference library table with a validation column marking metabolites passing the thresholds
- Counted number of validated metabolites (expected 571 for the HILIC LC-MS platform) and a match/mismatch comparison result written to library_count_verification.csv

## How to apply

Read each standard mixture replicate (the LC/MS method evaluates each standard mixture three times) as mzXML/mzML, generate standard analytes' EICs, and apply IRLS baseline correction plus Savitzky-Golay smoothing. For each peak compute MetaMOPE's eight metrics: MCQ, maximum intensity, RSD of retention time, standard-over-blank (Std/Blk) intensity ratio, jaggedness, asymmetry factor, FWHM and modality. Record a peak as valid only when Std/Blk exceeds a user-specified threshold r, the RT RSD across the three replicates is below threshold c, and peak intensity exceeds threshold I. Use the resulting validation column to decide whether each metabolite in the sample mixture is validated, then count the validated rows and compare against the reported 571 metabolites for the HILIC LC-MS platform; a match confirms the reproduction.

## Related tools

- **MetaMOPE** (Computes the eight peak metrics (MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) from EICs and sets the validation column via user-specified thresholds) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Containerizes and deploys the MetaMOPE web tool used to run the validation workflow) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source code of the multiple validation metrics used by MetaMOPE) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build
```

## Evaluation signals

- The counted number of validated metabolite rows equals 571 for the HILIC LC-MS platform
- Every validated peak satisfies Std/Blk > r, RT RSD < c, and intensity > I, as encoded in the validation column
- Metrics MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM and modality are all populated for each retained peak
- Retention time of each reference analyte is consistent across the three replicate EICs (RSD below threshold c)
- library_count_verification.csv records the counted total and an explicit match/mismatch against 571

## Limitations

- Validation depends on user-specified thresholds (r, c, I), so results are not absolute and vary with threshold choice
- The workflow was validated with 20 chemical standards; performance on other standard sets is not established in the article
- No changelog or discussion content was available, so reproducibility gaps and missing artifacts could not be extracted from the document

## Evidence

- [abstract] Filter criteria for validated peaks: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
- [abstract] Eight peak metrics: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [abstract] Validation column semantics: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [results] 571-metabolite reference library result: "This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform"
- [readme] Source code acquisition for the library: "Run `git clone git@github.com:CMDM-Lab/MetaHILIC.git` to get all source code"
