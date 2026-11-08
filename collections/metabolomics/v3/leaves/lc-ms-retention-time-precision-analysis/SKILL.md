---
name: lc-ms-retention-time-precision-analysis
description: Use when replicate LC-MS runs of chemical standards are available (e.g.
  each standard mixture evaluated three times) and you need to quantify retention
  time reproducibility or filter peaks by an RSD threshold before accepting a metabolite
  as validated.
when_to_use_negative:
- Only a single non-replicate run is available — RT RSD cannot be computed and the
  workflow degrades to the other per-peak metrics
- The goal is mobile-phase optimization (peak quality score from Sp and Saf) rather
  than replicate retention-time precision assessment
- Input is already a processed peak/feature table with retention times rather than
  raw mzXML/mzML chromatogram data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaMOPE
  role: Containerized web tool that batch-evaluates chromatography peaks across replicate
    runs, determines retention times from EICs, and generates the RSD-of-retention-time
    metric
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source-code functions for the per-peak metrics, including replicate-based
    retention-time RSD, exported by MetaMOPE for reuse
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment container for running the MetaMOPE workflow locally via docker-compose
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:451bbcaf453a999b0da32faa457acd7b83eb7edca88c66d8ff34998d0d7e8e03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-ms-retention-time-precision-analysis

## Summary

Assess chromatographic retention time precision in LC-MS metabolomics by evaluating EIC-derived peak retention times across replicate injections and computing the relative standard deviation (RSD) of retention time as one of MetaMOPE's eight per-peak quality metrics.

## When to use

Use when replicate LC-MS runs of chemical standards are available (e.g. each standard mixture evaluated three times) and you need to quantify retention time reproducibility or filter peaks by an RSD threshold before accepting a metabolite as validated.

## When NOT to use

- Only a single non-replicate run is available — RT RSD cannot be computed and the workflow degrades to the other per-peak metrics
- The goal is mobile-phase optimization (peak quality score from Sp and Saf) rather than replicate retention-time precision assessment
- Input is already a processed peak/feature table with retention times rather than raw mzXML/mzML chromatogram data

## Inputs

- Replicate mzXML or mzML LC-MS files for each standard mixture (e.g. three replicate injections)
- Optionally, blank mzXML/mzML files for Std/Blk intensity-ratio computation
- List of standard analytes with target m/z values and m/z window width
- User-specified thresholds: RSD of retention time (c), Std/Blk ratio (r), and minimum peak intensity (I)

## Outputs

- Per-peak RSD of retention time across the three replicates
- Retention time of each reference analyte derived from EICs of the replicates
- Eight-metric quality table per analyte (MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) written to CSV
- Validation column indicating whether each metabolite passes the user-specified thresholds

## How to apply

Extract the EIC for each standard analyte from the input mzXML/mzML file using the specified m/z window, then apply IRLS baseline correction and Savitzky–Golay smoothing to each EIC. Detect peaks in the smoothed profile and record the retention time of the local maximum for each analyte in each of the three replicates; the workflow automatically processes repeats and determines retention time of large numbers of standards. Compute the RSD of the retention time across replicates (RSD = SD/mean) for each detected peak. Retain a peak only if its retention-time RSD is below a user-specified threshold c, together with the companion criteria that Std/Blk intensity ratio exceeds threshold r and peak intensity exceeds threshold I. Peaks passing all thresholds are flagged in the validation column of the metrics table; replicate information is otherwise omitted when no replicate files are supplied.

## Related tools

- **MetaMOPE** (Containerized web tool that batch-evaluates chromatography peaks across replicate runs, determines retention times from EICs, and generates the RSD-of-retention-time metric) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source-code functions for the per-peak metrics, including replicate-based retention-time RSD, exported by MetaMOPE for reuse) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container for running the MetaMOPE workflow locally via docker-compose) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; cp .env.example .env; docker-compose up --build
```

## Evaluation signals

- Each analyte's retention time is reported consistently across all three replicate injections (low scatter) before RSD is computed
- Computed RT RSD values fall below the user-specified threshold c for peaks marked validated in the output CSV
- The output metrics table contains all eight columns (MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) with one row per analyte
- Peaks failing any of the three thresholds (Std/Blk > r, RT RSD < c, intensity > I) appear with a negative validation flag
- Retention times determined from replicate EICs can be used to build the reference library, as demonstrated for the 571-metabolite HILIC LC-MS library

## Limitations

- RSD of retention time requires replicate files; when no replicates are supplied this metric (and Std/Blk) cannot be computed
- Thresholds r, c, and I are user-specified, so validation outcomes are parameter-dependent and not fixed by the tool
- The workflow was developed and validated for HILIC LC-MS of polar metabolites with chemical standards; applicability to other chromatographic modes is not established

## Evidence

- [abstract] The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates.: "The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates."
- [abstract] Generate eight metrics for each peak: MCQ, the maximum intensity, the RSD of retention time, the ratio of standard over blank (Std/Blk) intensity, jaggedness, asymmetry factor, FWHM and modality.: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [abstract] Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation (RSD) of the retention time of the detected peaks of the three replicates is less than a user-specified threshold c, and the peak intensity is larger than a user-specified threshold I.: "the relative standard deviation (RSD) of the retention time of the detected peaks of the three replicates is less than a user-specified threshold c"
- [readme] Moreover, the workflow enables one to automatically process the repeats by evaluating chromatography peaks and determining the retention time of large numbers of standards.: "the workflow enables one to automatically process the repeats by evaluating chromatography peaks and determining the retention time of large numbers of standards"
- [results] This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform.: "This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform"
