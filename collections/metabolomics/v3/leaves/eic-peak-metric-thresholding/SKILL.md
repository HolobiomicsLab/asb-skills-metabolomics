---
name: eic-peak-metric-thresholding
description: Use when constructing a retention-time reference library from replicate
  LC-MS injections of chemical standard mixtures and you need to decide, per metabolite,
  whether its EIC peak passes user-specified Std/Blk, retention-time RSD, and intensity
  thresholds.
when_to_use_negative:
- Input is raw chromatographic data not yet processed into MetaMOPE's per-peak metrics
  — EIC generation, IRLS baseline correction, and Savitzky-Golay smoothing must happen
  first via MetaMOPE.
- The goal is choosing an optimal mobile phase rather than validating standard peaks
  and building a retention-time library — use MetaMOPE's peak quality score (Sp +
  Saf) workflow instead.
- Only one replicate injection is available — the retention-time RSD criterion (threshold
  c) requires three replicate injections.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaMOPE
  role: Containerized web tool that generates the eight per-peak metrics (MCQ, maximum
    intensity, retention-time RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM,
    modality) from EICs and applies the threshold filters
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Container runtime used to deploy and run MetaMOPE locally via docker-compose
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source functions MetaMOPE provides for computing the peak
    metrics
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:d823513bbd563530e648eb533b0f808e9986f3d9f4fee7c0590717bf1d112e5b
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
yes_rate: 1.0
n_claims: 10
---

# eic-peak-metric-thresholding

## Summary

Validates metabolite peaks from extracted ion chromatograms (EICs) by thresholding MetaMOPE's eight per-peak metrics — Std/Blk intensity ratio (threshold r), retention-time RSD (threshold c), and peak intensity (threshold I) — to flag validated metabolites and build retention-time reference libraries.

## When to use

Use when constructing a retention-time reference library from replicate LC-MS injections of chemical standard mixtures and you need to decide, per metabolite, whether its EIC peak passes user-specified Std/Blk, retention-time RSD, and intensity thresholds.

## When NOT to use

- Input is raw chromatographic data not yet processed into MetaMOPE's per-peak metrics — EIC generation, IRLS baseline correction, and Savitzky-Golay smoothing must happen first via MetaMOPE.
- The goal is choosing an optimal mobile phase rather than validating standard peaks and building a retention-time library — use MetaMOPE's peak quality score (Sp + Saf) workflow instead.
- Only one replicate injection is available — the retention-time RSD criterion (threshold c) requires three replicate injections.

## Inputs

- Per-peak metric tables from MetaMOPE for three replicate LC-MS injections of each chemical standard mixture (eight metrics: MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, modality)
- Input mzXML/mzML files of standard mixtures and blanks from which EICs are generated
- User-specified thresholds r (Std/Blk ratio), c (retention-time RSD), and I (peak intensity)

## Outputs

- Retention-time reference library table with a validation column and validated retention time per metabolite
- Per-metabolite validation flags determined by the user-specified thresholds

## How to apply

Load the eight per-peak metrics MetaMOPE generates for each peak (MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, modality) computed from EICs of the three replicate injections of each standard mixture, after IRLS baseline correction and Savitzky-Golay smoothing. Record a peak only if its Std/Blk intensity ratio exceeds user-specified threshold r, the retention-time RSD across the three replicates is below user-specified threshold c, and peak intensity exceeds user-specified threshold I. Mark each metabolite in the validation column according to whether it passes these thresholds; note that thresholds are user-chosen, not fixed defaults, so they must be supplied per analysis. For validated metabolites, extract the retention time determined from the EICs of the three replicates and write a retention-time reference library table (validation flag plus validated retention time per metabolite), matching the 571-metabolite reference-library format produced for the HILIC LC-MS platform.

## Related tools

- **MetaMOPE** (Containerized web tool that generates the eight per-peak metrics (MCQ, maximum intensity, retention-time RSD, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality) from EICs and applies the threshold filters) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Container runtime used to deploy and run MetaMOPE locally via docker-compose) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source functions MetaMOPE provides for computing the peak metrics) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build  # then submit mzXML/mzML replicates + thresholds r, c, I via http://localhost:13002
```

## Evaluation signals

- Every retained peak satisfies all three criteria: Std/Blk ratio > r, retention-time RSD < c, and intensity > I.
- Each metabolite's validation column correctly reflects whether its peak passed the user-specified thresholds.
- Output table follows the 571-metabolite reference-library format: validation flag plus validated retention time per metabolite.
- Retention times of validated metabolites are consistent across the three replicate EICs (RSD below threshold c).

## Limitations

- Thresholds r, c, and I are user-specified, not fixed defaults, so validation outcomes depend on analyst choices.
- The three-criterion filter applies only to peaks whose metrics were already generated by MetaMOPE; no discussion-section caveats were available from the document.
- Requires replicate injections (three replicates per standard mixture) for the RSD criterion to be computable.

## Evidence

- [abstract] Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation"
- [abstract] The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds.: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [results] This workflow was successfully applied to construct a reference library of 571 metabolites: "This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform"
- [results] the workflow enables one to automatically process the repeats by evaluating chromatography peaks and determining the retention time of large numbers of standards: "the workflow enables one to automatically process the repeats by evaluating chromatography peaks and determining the retention time of large numbers of standards"
- [readme] README deployment instructions for running MetaMOPE locally: "Run `docker-compose up --build`"
