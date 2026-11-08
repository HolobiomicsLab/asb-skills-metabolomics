---
name: irls-baseline-correction
description: Use when raw mzXML/mzML LC-MS data must be converted into per-analyte
  EICs whose baselines drift, and peak metrics (MCQ, maximum intensity, asymmetry
  factor, FWHM) must be computed on a corrected signal.
when_to_use_negative:
- Input is already a baseline-corrected, smoothed peak table or feature table rather
  than raw mzXML/mzML chromatographic data
- The goal is mobile-phase selection scoring (peak quality score Sp + Saf) rather
  than per-EIC baseline correction and metric computation
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaMOPE
  role: Containerized web tool whose peak evaluation workflow applies IRLS baseline
    correction to EICs before computing quality metrics
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions for the multiple peak-quality metrics,
    including baseline correction and smoothing routines
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment mechanism; run the MetaMOPE stack with docker-compose to access
    the workflow locally
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:a437c65866715c525a3c7df3b5997357d34e59d6e5ad57c3cc56ef56d878be05
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# irls-baseline-correction

## Summary

Baseline-correct extracted ion chromatograms (EICs) using Iteratively Reweighted Least Squares (IRLS) before Savitzky-Golay smoothing and peak metric computation in LC-MS metabolomics peak evaluation, as implemented in the MetaMOPE workflow.

## When to use

Use when raw mzXML/mzML LC-MS data must be converted into per-analyte EICs whose baselines drift, and peak metrics (MCQ, maximum intensity, asymmetry factor, FWHM) must be computed on a corrected signal.

## When NOT to use

- Input is already a baseline-corrected, smoothed peak table or feature table rather than raw mzXML/mzML chromatographic data
- The goal is mobile-phase selection scoring (peak quality score Sp + Saf) rather than per-EIC baseline correction and metric computation

## Inputs

- mzXML or mzML LC-MS data file
- List of standard analytes with target m/z values and m/z window for EIC extraction
- Optional replicate and blank files for RSD of retention time and Std/Blk metrics

## Outputs

- Baseline-corrected EICs (intermediate)
- Savitzky-Golay smoothed EICs (intermediate)
- Per-peak table of eight metrics (MCQ, maximum intensity, RSD of retention time, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) with a validation column, exported to CSV

## How to apply

Read the input mzXML/mzML file and generate each chemical standard's EIC using the specified m/z window. Apply IRLS (Iteratively Reweighted Least Squares) baseline correction to each EIC to remove the drifting baseline, then pass the baseline-corrected EIC through a Savitzky-Golay filter for smoothing. Only after these two preprocessing steps should peak regions be detected and metrics such as the maximum intensity, mass chromatographic quality (MCQ) index, asymmetry factor, FWHM, jaggedness, and modality be calculated. The rationale is that baseline drift inflates local maximum intensity and distorts the asymmetry factor and MCQ, so correction and smoothing precede all metric computation in the MetaMOPE pipeline. The eight per-peak metrics are then aggregated into a metrics table with a validation column governed by user-specified thresholds (Std/Blk ratio r, RSD of retention time c, intensity I).

## Related tools

- **MetaMOPE** (Containerized web tool whose peak evaluation workflow applies IRLS baseline correction to EICs before computing quality metrics) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions for the multiple peak-quality metrics, including baseline correction and smoothing routines) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment mechanism; run the MetaMOPE stack with docker-compose to access the workflow locally) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && cp .env.example .env && docker-compose up --build  # then submit mzXML/mzML via http://localhost:13002
```

## Evaluation signals

- Smoothed, baseline-corrected EIC shows baseline near zero intensity outside the peak region, with the local maximum intensity attributable to the analyte peak rather than drift
- MCQ and asymmetry factor computed post-correction fall in reasonable ranges; asymmetry factor should be near 1 for symmetric peaks
- Per-analyte output table contains all eight metrics (MCQ, max intensity, RSD of RT, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) plus the validation column
- Retention time determined from the corrected EIC is consistent across the three replicate injections of each standard mixture
- Metrics match those reported by the MetaMOPE web service (https://metamope.cmdm.tw) for the same input file and thresholds

## Limitations

- Thresholds (Std/Blk r, RSD c, intensity I) are user-specified, so validation of metabolites in the sample mixture depends on the chosen cutoffs
- Metrics such as RSD of retention time and Std/Blk intensity ratio are only computed if replicate and blank files are supplied
- No changelog/discussion content was available, so reproducibility gaps and edge-case behavior of the IRLS correction cannot be assessed from the article

## Evidence

- [abstract] The extracted ion chromatograms of each chemical standard were retrieved, followed by baseline correction and smoothing of EIC using the IRLS algorithm and Savitzky-Golay filter.: [evidence span withheld — anchor status: not_found]
- [results] Generate eight metrics for each peak: MCQ, maximum intensity, RSD of retention time, Std/Blk intensity, jaggedness, asymmetry factor, FWHM and modality.: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [abstract] Read input mzXML/mzML and generate standard analytes' EIC.: [evidence span withheld — anchor status: not_found]
- [results] A mass chromatographic quality value, asymmetric factor, and local maximum intensity of the EIC were calculated.: "A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to determine the number of peaks and peak retention time"
