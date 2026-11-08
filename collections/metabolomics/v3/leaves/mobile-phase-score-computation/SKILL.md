---
name: mobile-phase-score-computation
description: Use when comparing two or more HILIC LC-MS mobile phases using replicate
  mzXML/mzML acquisitions of a chemical-standard mixture and needing an objective,
  metric-based choice of the optimal phase.
when_to_use_negative:
- Input is already a validated peak/feature table rather than raw mzXML/mzML replicate
  acquisitions of standard mixtures
- The goal is metabolite annotation or building the reference library itself rather
  than choosing among candidate mobile phases
- Only a single mobile phase is available with no comparison to be made — there is
  nothing to rank by Sp + Saf
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaMOPE
  role: Containerized web tool that batch-evaluates chromatography peaks, computes
    the eight peak metrics and the Sp/Saf peak quality score used to rank mobile phases
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: 'Deployment container: run docker-compose up --build to serve MetaMOPE locally
    for the mobile-phase comparison workflow'
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source code of functions for the multiple peak-evaluation metrics
    (MCQ, asymmetry factor, etc.) underlying the score computation
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:68cc540a95f2a5b3a32f2e09b8c6be04a42abb7142083e43df24be088a85f695
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
n_claims: 7
---

# mobile-phase-score-computation

## Summary

Compute the MetaMOPE peak quality score (Sp + Saf) for each candidate LC-MS mobile phase by batch-evaluating chromatography peaks from standard-mixture replicates, then select the phase with the highest total score as the optimized mobile phase.

## When to use

Use when comparing two or more HILIC LC-MS mobile phases using replicate mzXML/mzML acquisitions of a chemical-standard mixture and needing an objective, metric-based choice of the optimal phase.

## When NOT to use

- Input is already a validated peak/feature table rather than raw mzXML/mzML replicate acquisitions of standard mixtures
- The goal is metabolite annotation or building the reference library itself rather than choosing among candidate mobile phases
- Only a single mobile phase is available with no comparison to be made — there is nothing to rank by Sp + Saf

## Inputs

- Replicate mzXML/mzML files per mobile phase (three LC-MS acquisitions of the chemical-standards mixture per phase)
- List of standard analytes (m/z and identifiers) for EIC generation
- User-specified validation thresholds: peak maximum intensity, MCQ, Std/Blk ratio r, retention-time RSD c

## Outputs

- Peak quality score table per mobile phase (resolved-peak count, Sp, Saf, total score), e.g. mobile_phase_peak_quality_scores.csv
- Selected optimal mobile phase (highest Sp + Saf)

## How to apply

Read the replicate mzXML/mzML files for each mobile phase (e.g. three replicates per phase of a 20-chemical-standard mixture), generate extracted ion chromatograms (EICs) for the standard analytes, and preprocess each EIC with IRLS baseline correction followed by Savitzky-Golay smoothing. For every analyte peak compute the eight MetaMOPE metrics (MCQ, maximum intensity, retention-time RSD, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, modality) and count validated peaks — those whose maximum intensity and MCQ exceed user-specified thresholds and that are resolved. From the validated-peak counts compute the detected-peaks score Sp for each phase and the averaged asymmetry factor of the commonly validated peaks (Saf, assessed via one-way ANOVA), then sum Sp + Saf into the final peak quality score. Select the mobile phase with the highest peak quality score; in the article's HILIC example, Mobile Phase B (10 resolved peaks, Sp = 2, Saf = 1, total 3) outranked Phase A (4 resolved peaks, Sp = 1, Saf = 1, total 2).

## Related tools

- **MetaMOPE** (Containerized web tool that batch-evaluates chromatography peaks, computes the eight peak metrics and the Sp/Saf peak quality score used to rank mobile phases) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container: run docker-compose up --build to serve MetaMOPE locally for the mobile-phase comparison workflow) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source code of functions for the multiple peak-evaluation metrics (MCQ, asymmetry factor, etc.) underlying the score computation) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; cp .env.example .env; docker-compose up --build  # then submit replicate mzXML/mzML files per mobile phase at http://localhost:13002
```

## Evaluation signals

- Phase A vs Phase B reference check: Phase A yields 4 resolved peaks (Sp = 1), Phase B yields 10 resolved peaks (Sp = 2), Saf = 1 for both phases
- Final scores match the reported totals: Phase A = 2, Phase B = 3, with Mobile Phase B selected as optimal
- Only peaks whose maximum intensity and MCQ exceed user-specified thresholds are counted as validated/resolved peaks
- The output table (e.g. mobile_phase_peak_quality_scores.csv) contains resolved-peak count, Sp, Saf, and summed peak quality score per phase

## Limitations

- The validation column depends on user-specified thresholds (peak intensity I, Std/Blk ratio r, retention-time RSD c), so results are threshold-sensitive
- No changelog/discussion content was available, so reproducibility gaps and limitations could not be extracted from that section

## Evidence

- [abstract] The optimal mobile phase can be quickly determined by selecting the mobile phase that produces the largest number of resolved peaks.: "The optimal mobile phase can be quickly determined by selecting the mobile phase that produces the largest number of resolved peaks."
- [abstract] Final peak quality score is the sum of Sp and Saf: "The final peak quality score is the sum of the score of detected peaks (Sp) and the score of the asymmetry factor (Saf)."
- [abstract] Peak metrics computed after IRLS baseline correction and Savitzky-Golay smoothing: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [abstract] Validation filter criteria for recorded peaks: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r, the relative standard deviation"
- [readme] README deployment instructions for MetaMOPE: "Run `docker-compose up --build`"
