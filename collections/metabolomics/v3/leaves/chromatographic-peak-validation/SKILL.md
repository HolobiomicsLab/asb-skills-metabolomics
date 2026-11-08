---
name: chromatographic-peak-validation
description: Use when replicate LC/MS injections of chemical standard mixtures (mzXML/mzML
  with blank runs) must be filtered into validated peaks to build a retention-time
  reference library, e.g. the HILIC LC-MS platform's 571-metabolite library.
when_to_use_negative:
- Input is already a validated peak/feature table — re-running the Std/Blk > r, RSD
  < c, intensity > I filters would double-filter and misassign validation flags.
- The goal is de novo peak picking on untargeted samples without chemical-standard
  replicates and accompanying blank runs, since Std/Blk and RT RSD criteria cannot
  be computed.
- Assessing mobile-phase peak quality end-to-end rather than validating individual
  metabolite peaks for a reference library (use the mobile-phase-scoring skill instead).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaMOPE
  role: Containerized web tool whose workflow computes the eight per-peak metrics
    and applies the Std/Blk, RT-RSD and intensity thresholds to mark metabolites validated
    and extract retention times
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Container runtime used to deploy MetaMOPE locally via docker-compose so the
    validation workflow is reproducible
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions implementing the peak-evaluation metrics
    (MCQ, asymmetry factor, etc.)
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:1d8d44a7e5a275cfa456e89b3e7ed07e769df1ea54b740d79c91e9b645c2be24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chromatographic-peak-validation

## Summary

Validates metabolite peaks detected from LC-MS standard-mixture replicates using MetaMOPE's eight per-peak metrics (MCQ, maximum intensity, RT RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality) against user-specified thresholds (r, c, I), producing validation flags and validated retention times for a reference library.

## When to use

Use when replicate LC/MS injections of chemical standard mixtures (mzXML/mzML with blank runs) must be filtered into validated peaks to build a retention-time reference library, e.g. the HILIC LC-MS platform's 571-metabolite library.

## When NOT to use

- Input is already a validated peak/feature table — re-running the Std/Blk > r, RSD < c, intensity > I filters would double-filter and misassign validation flags.
- The goal is de novo peak picking on untargeted samples without chemical-standard replicates and accompanying blank runs, since Std/Blk and RT RSD criteria cannot be computed.
- Assessing mobile-phase peak quality end-to-end rather than validating individual metabolite peaks for a reference library (use the mobile-phase-scoring skill instead).

## Inputs

- mzXML/mzML replicate files (three replicates) of a chemical-standards mixture acquired on the LC-MS platform
- Blank-run intensity data for Std/Blk ratio computation
- Extracted ion chromatograms (EICs) of standard analytes after IRLS baseline correction and Savitzky-Golay smoothing
- User-specified thresholds: r (Std/Blk ratio), c (RT RSD), I (peak intensity)

## Outputs

- Eight-metric table per peak (MCQ, maximum intensity, RT RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality)
- Validation flag per metabolite (validated vs not) under thresholds r, c, I
- Validated retention time per metabolite derived from the three replicate EICs, formatted as the 571-metabolite reference library table

## How to apply

Preprocess each analyte's extracted ion chromatogram (EIC) with IRLS baseline correction followed by Savitzky-Golay smoothing, then compute MetaMOPE's eight per-peak metrics: MCQ, maximum intensity, retention-time RSD across the three replicates, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM and modality. Record a peak only if the ratio of peak intensity to blank intensity (Std/Blk) exceeds the user-specified threshold r, the RSD of retention time of the three replicates is below threshold c, and peak intensity exceeds threshold I; write this decision into the validation column for each metabolite. For validated metabolites, extract the retention time determined from the EICs of the three replicates and emit the reference-library table (validation flag plus validated retention time per metabolite) in the 571-metabolite library format. The same validated-peak counts (max intensity and MCQ above thresholds) additionally feed the Sp score when comparing mobile phases.

## Related tools

- **MetaMOPE** (Containerized web tool whose workflow computes the eight per-peak metrics and applies the Std/Blk, RT-RSD and intensity thresholds to mark metabolites validated and extract retention times) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Container runtime used to deploy MetaMOPE locally via docker-compose so the validation workflow is reproducible) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions implementing the peak-evaluation metrics (MCQ, asymmetry factor, etc.))

## Examples

```
docker-compose up --build  # then open http://localhost:13002 to run MetaMOPE peak validation on replicate mzXML/mzML inputs
```

## Evaluation signals

- Validation column agrees with the recorded filter: Std/Blk ratio > r AND retention-time RSD < c AND peak intensity > I for every marked-validated metabolite.
- Every retained metabolite has a retention time consistent with its EIC across all three replicate injections.
- Output table matches the 571-metabolite HILIC reference-library format (validation flag plus validated retention time per metabolite).
- On the 20-standard benchmark, validated (resolved) peak counts reproduce Phase A = 4 and Phase B = 10, i.e. Sp = 1 vs Sp = 2 with Saf = 1 each.
- Per-peak metric table contains all eight metrics with plausible ranges (e.g. MCQ and maximum intensity above user thresholds only for validated peaks).

## Limitations

- Thresholds r, c and I are user-specified; the article gives no default cutoffs, so validation outcomes depend directly on chosen values.
- No discussion content was available in the document (no changelog found), so no reported limitations, missing artifacts, or reproducibility gaps could be extracted.
- The workflow presupposes three replicate injections of each standard mixture plus blank runs; it was validated only on the HILIC LC-MS platform setup.

## Evidence

- [abstract] Filter criteria r, c, I: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
- [abstract] Validation column semantics: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [abstract] Eight per-peak metrics: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [abstract] Validated-peak counting for mobile-phase scoring: "we computed the number of validated peaks whose maximum intensity and MCQ are the above user-specified threshold in the experiments of each mobile phase"
- [readme] Peak metrics used to determine retention time: "A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to"
