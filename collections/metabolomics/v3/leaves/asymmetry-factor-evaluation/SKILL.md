---
name: asymmetry-factor-evaluation
description: Use when comparing LC-MS mobile phases (e.g. HILIC phases A and B on
  20-chemical-standard mixtures) and you need peak shape quality — the averaged asymmetry
  factor of commonly validated peaks — as part of a peak quality score.
when_to_use_negative:
- When peak detection counts (Sp) alone suffice and peak-shape quality is not being
  compared between mobile phases
- When the input is already a processed peak table without the raw EIC signal — IRLS
  baseline correction and Savitzky-Golay smoothing must be applied to the extracted
  ion chromatograms first
- When chromatography is not LC-MS-based metabolomics (e.g. GC-MS or non-chromatographic
  spectra), since MetaMOPE's metrics and thresholds are designed for LC-MS EICs
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaMOPE
  role: Containerized web tool whose batch peak-evaluation workflow computes the asymmetry
    factor, averages it over commonly validated peaks (via one-way ANOVA), and folds
    it into the Saf score for mobile-phase ranking
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides the source-code functions for MetaMOPE's peak metrics, including
    the asymmetry factor calculation
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment container for MetaMOPE (docker-compose up --build) so the asymmetry-factor
    workflow can be run locally at http://localhost:13002
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:7766f9b280411e9bc47381678ab20d5f06b434ba68f1bebcefa8c94080749680
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

# asymmetry-factor-evaluation

## Summary

Computes the chromatographic peak asymmetry factor from extracted ion chromatograms (EICs) after IRLS baseline correction and Savitzky-Golay smoothing, and aggregates it into MetaMOPE's asymmetry-factor score (Saf) used to rank mobile phases for LC-MS metabolomics.

## When to use

Use when comparing LC-MS mobile phases (e.g. HILIC phases A and B on 20-chemical-standard mixtures) and you need peak shape quality — the averaged asymmetry factor of commonly validated peaks — as part of a peak quality score.

## When NOT to use

- When peak detection counts (Sp) alone suffice and peak-shape quality is not being compared between mobile phases
- When the input is already a processed peak table without the raw EIC signal — IRLS baseline correction and Savitzky-Golay smoothing must be applied to the extracted ion chromatograms first
- When chromatography is not LC-MS-based metabolomics (e.g. GC-MS or non-chromatographic spectra), since MetaMOPE's metrics and thresholds are designed for LC-MS EICs

## Inputs

- mzXML/mzML replicate files of chemical-standard mixtures acquired under each candidate mobile phase (e.g. three replicates per phase)
- List of standard analytes with m/z values for EIC extraction
- User-specified validation thresholds: Std/Blk intensity ratio (r), retention-time RSD (c), minimum peak intensity (I)

## Outputs

- Per-peak asymmetry factor among the eight MetaMOPE peak metrics (alongside MCQ, maximum intensity, retention-time RSD, Std/Blk ratio, jaggedness, FWHM, modality)
- Averaged asymmetry factor of commonly validated peaks and its score (Saf) per mobile phase
- Peak quality score table (mobile_phase_peak_quality_scores.csv) with Sp, Saf, and total score per mobile phase

## How to apply

Read the replicate mzXML/mzML files and generate standard analytes' EICs, then preprocess each EIC with IRLS baseline correction followed by Savitzky-Golay smoothing before calculating the asymmetry factor for every analyte peak. Validate peaks against user-specified thresholds (Std/Blk ratio r, retention-time RSD c across three replicates, and minimum intensity I); the asymmetry factor is then averaged over the peaks commonly validated across the compared mobile phases, using one-way analysis of variance. Convert the averaged asymmetry factor into the score Saf (in the MetaMOPE example, Saf = 1 for both Mobile Phase A and B) and combine it with the detected-peaks score Sp (Phase A: 4 resolved peaks -> Sp = 1; Phase B: 10 resolved peaks -> Sp = 2) to obtain the final peak quality score, summed into mobile_phase_peak_quality_scores.csv. The mobile phase with the highest Sp + Saf total (Mobile Phase B, score 3) is selected as optimal.

## Related tools

- **MetaMOPE** (Containerized web tool whose batch peak-evaluation workflow computes the asymmetry factor, averages it over commonly validated peaks (via one-way ANOVA), and folds it into the Saf score for mobile-phase ranking) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides the source-code functions for MetaMOPE's peak metrics, including the asymmetry factor calculation) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container for MetaMOPE (docker-compose up --build) so the asymmetry-factor workflow can be run locally at http://localhost:13002) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git; docker-compose up --build  # then access http://localhost:13002 to batch-evaluate EIC asymmetry factors across mobile phases
```

## Evaluation signals

- The mobile_phase_peak_quality_scores.csv table reproduces the article's example values: Phase A with 4 resolved peaks, Sp = 1, Saf = 1; Phase B with 10 resolved peaks, Sp = 2, Saf = 1; Phase B total score 3 (highest).
- Asymmetry factors are computed only after IRLS baseline correction and Savitzky-Golay smoothing of each EIC — scores computed on raw EICs indicate incorrect preprocessing.
- The averaged asymmetry factor is computed over commonly validated peaks (those passing max intensity and MCQ thresholds) across the compared phases, not over all detected peaks.
- Final peak quality score equals Sp + Saf per phase, and the selected optimal phase is the one with the largest number of resolved peaks in the worked example (Mobile Phase B).
- Per-peak output contains all eight MetaMOPE metrics (MCQ, maximum intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality), with the validation column reflecting user-specified thresholds.

## Limitations

- The asymmetry factor is one of several thresholds-dependent metrics: peak validation depends on user-specified Std/Blk ratio (r), retention-time RSD (c), and intensity (I) cutoffs, so Saf is sensitive to threshold choices.
- No discussion content was available from the article, so no additional limitations, failure modes, or reproducibility gaps could be extracted.
- The workflow requires three replicate acquisitions per mobile phase, since retention-time RSD across replicates is part of peak validation.

## Evidence

- [abstract] IRLS baseline correction and Savitzky-Golay smoothing precede asymmetry factor calculation: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [abstract] Saf is the averaged asymmetry factor of commonly validated peaks via one-way ANOVA: "we computed the number of validated peaks whose maximum intensity and MCQ are the above user-specified threshold in the experiments of each mobile phase, as well as the averaged asymmetry"
- [abstract] Final peak quality score is the sum of Sp and Saf: "The final peak quality score is the sum of the score of detected peaks (Sp) and the score of the asymmetry factor (Saf)."
- [abstract] Asymmetry factor is one of eight peak metrics generated by MetaMOPE: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity, and"
- [readme] MetaMOPE uses mass chromatographic quality value, asymmetry factor and local maximum intensity to determine peaks: "A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to determine the number of peaks and peak retention time."
