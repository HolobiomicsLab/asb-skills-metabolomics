---
name: standard-over-blank-intensity-ratio-analysis
description: Use when three-replicate LC-MS runs of chemical standards (mzML/mzXML)
  must be screened for blank contamination and reproducibility, i.e. deciding which
  metabolites are validated or excluded by Std/Blk, RSD, and intensity cutoffs.
when_to_use_negative:
- Peak-intensity or retention-time data are not yet available — the Std/Blk ratio
  must be computed after EIC extraction, IRLS baseline correction, and Savitzky-Golay
  smoothing
- The goal is mobile-phase selection rather than per-metabolite validation into the
  reference library
- Only a single run (no replicates) is available, since the retention-time RSD criterion
  requires three replicate measurements
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMOPE
  role: Containerized web tool that generates the EICs, computes the eight peak metrics
    including Std/Blk intensity ratio, and applies the r/c/I thresholds to emit the
    validation column
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source code of functions for the multiple peak metrics, including
    the Std/Blk ratio used in validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:7a3c6a5435624ba9ad00ed2d290080d9a0c9c5040982069a91decb92e1dfe021
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - MetaMOPE
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# standard-over-blank-intensity-ratio-analysis

## Summary

Computes the standard-over-blank (Std/Blk) peak intensity ratio from extracted ion chromatograms in MetaMOPE, using it with retention-time RSD and peak-intensity thresholds to decide whether an LC-MS metabolite standard is validated into a reference library.

## When to use

Use when three-replicate LC-MS runs of chemical standards (mzML/mzXML) must be screened for blank contamination and reproducibility, i.e. deciding which metabolites are validated or excluded by Std/Blk, RSD, and intensity cutoffs.

## When NOT to use

- Peak-intensity or retention-time data are not yet available — the Std/Blk ratio must be computed after EIC extraction, IRLS baseline correction, and Savitzky-Golay smoothing
- The goal is mobile-phase selection rather than per-metabolite validation into the reference library
- Only a single run (no replicates) is available, since the retention-time RSD criterion requires three replicate measurements

## Inputs

- Three replicate LC-MS runs of chemical standards as mzXML/mzML files
- User-specified thresholds: Std/Blk ratio r (e.g. 6), retention-time RSD c (e.g. 15%), peak intensity I (e.g. 1000)

## Outputs

- Per-peak metrics table with MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, and modality
- Per-metabolite validation flag column indicating which standards are validated into the reference library

## How to apply

Read the sample's input mzXML/mzML file and generate the standard analytes' EICs, then preprocess each EIC with IRLS baseline correction and Savitzky-Golay smoothing before computing metrics. For each standard peak, compute the Std/Blk intensity ratio (threshold r, e.g. r=6), the RSD of retention time across the three replicates (threshold c, e.g. c=15%), and the peak maximum intensity (threshold I, e.g. I=1000). Only peaks with Std/Blk > r, retention-time RSD < c, and maximum intensity > I are recorded as validated; e.g. in MetaMOPE's 20-standards test, taurocholate, l-aspartate, and sphingosine passed while estradiol was excluded for low Std/Blk (4.719 < 6) and l-cystine for low intensity (915.368 < 1000). The validation column in the output table records this per-metabolite decision, which is the basis for constructing the reference library (571 metabolites in the published HILIC LC-MS application).

## Related tools

- **MetaMOPE** (Containerized web tool that generates the EICs, computes the eight peak metrics including Std/Blk intensity ratio, and applies the r/c/I thresholds to emit the validation column) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source code of functions for the multiple peak metrics, including the Std/Blk ratio used in validation)

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; docker-compose up --build; then open http://localhost:13002 and apply thresholds r=6, c=15%, I=1000
```

## Evaluation signals

- Each validated standard must simultaneously satisfy Std/Blk > r, retention-time RSD < c, and maximum intensity > I; any single-criterion failure excludes the metabolite (e.g. estradiol at Std/Blk 4.719 < 6, l-cystine at intensity 915.368 < 1000)
- The output table contains exactly eight metrics per peak (MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) plus a validation column
- Reproducing the 20-standards benchmark should yield taurocholate, l-aspartate, and sphingosine as validated
- EICs must show evidence of IRLS baseline correction and Savitzky-Golay smoothing before metrics are computed

## Limitations

- Thresholds r, c, and I are user-specified, so validation decisions are threshold-dependent and must be reported explicitly
- The Std/Blk criterion requires blank runs alongside standard runs; without a blank the ratio cannot be computed
- No discussion content was available for this article, so additional failure modes could not be extracted from the source

## Evidence

- [abstract] Filter steps (candidate fine-grained sub-tasks): "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
- [abstract] Per-peak metrics: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [abstract] Validation column semantics: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [readme] Deployment / invocation: "Run `docker-compose up --build`"
