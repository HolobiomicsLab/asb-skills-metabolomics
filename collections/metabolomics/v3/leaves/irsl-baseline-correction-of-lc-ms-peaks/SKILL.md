---
name: irsl-baseline-correction-of-lc-ms-peaks
description: Use when raw mzXML/mzML metabolomics LC-MS runs must be converted into
  clean per-analyte EICs before computing peak quality metrics (maximum intensity,
  MCQ, asymmetry factor, FWHM) or applying intensity/RSD validation thresholds.
when_to_use_negative:
- Input is already a processed peak/feature table rather than raw mzXML/mzML chromatographic
  data
- The goal is mobile-phase selection requiring only counts of validated peaks where
  signals are already baseline-free
- Chromatograms contain only sharp isolated peaks with negligible baseline drift,
  where IRLS correction adds little value
tools:
- name: MetaMOPE
  role: Containerized web tool that reads mzXML/mzML, generates EICs, and applies
    IRLS baseline correction plus Savitzky-Golay smoothing before metric calculation
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Provides source functions for the multiple peak metrics written in R, usable
    alongside the correction workflow
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Runs the MetaMOPE stack via docker-compose so the correction workflow is reproducible
    locally
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:f7875c306571fcd49844377ba621f7006e61ac8851b4839dba4e13e86d794f0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# irsl-baseline-correction-of-lc-ms-peaks

## Summary

Preprocess extracted ion chromatograms (EICs) from mzML/mzXML LC-MS runs using IRLS (iteratively reweighted least squares) baseline correction followed by Savitzky-Golay smoothing, so that peak metrics such as maximum intensity, MCQ, and asymmetry factor are computed on drift-corrected, denoised signals in the MetaMOPE workflow.

## When to use

Use when raw mzXML/mzML metabolomics LC-MS runs must be converted into clean per-analyte EICs before computing peak quality metrics (maximum intensity, MCQ, asymmetry factor, FWHM) or applying intensity/RSD validation thresholds.

## When NOT to use

- Input is already a processed peak/feature table rather than raw mzXML/mzML chromatographic data
- The goal is mobile-phase selection requiring only counts of validated peaks where signals are already baseline-free
- Chromatograms contain only sharp isolated peaks with negligible baseline drift, where IRLS correction adds little value

## Inputs

- mzML or mzXML LC-MS run files (e.g. three replicate runs of 20 chemical standards)
- Standard analyte list defining m/z windows for extracted ion chromatogram (EIC) generation

## Outputs

- Baseline-corrected and Savitzky-Golay smoothed EICs per analyte
- Per-peak metrics computed on corrected EICs: MCQ, maximum intensity, RSD of retention time, Std/Blk ratio, jaggedness, asymmetry factor, FWHM, modality

## How to apply

Read each sample's input mzXML/mzML file and generate the standard analytes' EICs. Apply IRLS (Iteratively reweighted least squares) baseline correction to each EIC to remove low-frequency chromatographic drift, then apply the Savitzky-Golay filter to smooth the corrected trace. Compute downstream peak metrics — maximum intensity, mass chromatographic quality index (MCQ), and asymmetry factor — on the corrected and smoothed EICs, since these feed the MetaMOPE validation thresholds (e.g. Std/Blk ratio r=6, RSD of retention time c=15%, peak intensity I=1000). The correction must precede metric calculation because uncorrected baselines inflate maximum intensity and distort MCQ and asymmetry values.

## Related tools

- **MetaMOPE** (Containerized web tool that reads mzXML/mzML, generates EICs, and applies IRLS baseline correction plus Savitzky-Golay smoothing before metric calculation) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Provides source functions for the multiple peak metrics written in R, usable alongside the correction workflow) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Runs the MetaMOPE stack via docker-compose so the correction workflow is reproducible locally) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && cp .env.example .env && docker-compose up --build
```

## Evaluation signals

- EIC traces after correction show a flat baseline with smoothed peaks, and subsequent maximum intensity/MCQ/asymmetry values are computed on the corrected trace
- Downstream threshold application reproduces reference decisions: taurocholate, l-aspartate, sphingosine validated; estradiol excluded with Std/Blk 4.719 < r=6; l-cystine excluded with intensity 915.368 < I=1000
- Retention-time RSD across three replicates stays below the c=15% cutoff for validated analytes, confirming correction did not distort peak-picking
- All eight metrics (MCQ, maximum intensity, RSD of retention time, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) are emitted per peak without missing values for detected analytes

## Limitations

- No discussion content was available; the document states no changelog was found, so reproducibility gaps or parameter sensitivities of the IRLS correction cannot be extracted from this source
- Correction quality depends on correct EIC m/z windows; mis-specified standard analyte definitions propagate into the corrected traces and all derived metrics
- Peak validity still depends on user-specified thresholds (r, c, I), so a correct baseline correction alone does not guarantee a metabolite is validated

## Evidence

- [abstract] IRLS correction precedes metric computation: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [results] EIC retrieval and correction order in the validation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Input format and correction pipeline entry point: [evidence span withheld — anchor status: not_found]
- [abstract] Metric suite computed on corrected EICs: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [readme] Deployment route to run the correction workflow locally: "Run `docker-compose up --build`"
