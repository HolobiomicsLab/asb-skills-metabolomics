---
name: chromatographic-peak-quality-metrics
description: 'Use when evaluating LC-MS metabolomics peak quality for chemical standards:
  you have EICs from mzXML/mzML files (with optional replicate and blank runs) and
  need standardized per-peak quality metrics to validate or rank peaks.'
when_to_use_negative:
- When the goal is choosing the optimal mobile phase by comparing resolved-peak counts
  and asymmetry scores across mobile-phase experiments (use the MetaMOPE mobile-phase
  determination scoring workflow instead)
- When peak detection must be done directly on raw, uncorrected profiles — the metrics
  are computed only after IRLS baseline correction and Savitzky-Golay smoothing of
  the EIC
- When the input is already a processed feature table rather than raw mzXML/mzLC-MS
  chromatographic data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetaMOPE
  role: Containerized web tool and R source-code functions that generate the eight
    per-peak metrics (MCQ, maximum intensity, retention-time RSD, Std/Blk, jaggedness,
    asymmetry factor, FWHM, modality) from EICs
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions MetaMOPE provides for computing the
    multiple peak quality metrics
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment container for MetaMOPE, launched via docker-compose so the peak-evaluation
    web service runs locally
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:bcab46d2f1da1932f475a9e93513c251cfb2ba34ee3a18e832a0cf94a3502e2f
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chromatographic-peak-quality-metrics

## Summary

Computes the eight MetaMOPE per-peak chromatographic quality metrics (MCQ, maximum intensity, RSD of retention time, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) from extracted ion chromatograms in mzXML/mzML files, after IRLS baseline correction and Savitzky-Golay smoothing.

## When to use

Use when evaluating LC-MS metabolomics peak quality for chemical standards: you have EICs from mzXML/mzML files (with optional replicate and blank runs) and need standardized per-peak quality metrics to validate or rank peaks.

## When NOT to use

- When the goal is choosing the optimal mobile phase by comparing resolved-peak counts and asymmetry scores across mobile-phase experiments (use the MetaMOPE mobile-phase determination scoring workflow instead)
- When peak detection must be done directly on raw, uncorrected profiles — the metrics are computed only after IRLS baseline correction and Savitzky-Golay smoothing of the EIC
- When the input is already a processed feature table rather than raw mzXML/mzLC-MS chromatographic data

## Inputs

- Sample mzXML or mzML file containing LC-MS runs of chemical standard mixtures
- m/z window specification for generating each standard analyte's extracted ion chromatogram (EIC)
- Optional: three replicate LC-MS runs per standard mixture (for RSD of retention time)
- Optional: blank run files (for Std/Blk intensity ratio)
- User-specified thresholds r (Std/Blk), c (retention-time RSD), and I (peak intensity)

## Outputs

- Per-analyte table of eight peak metrics: MCQ, maximum intensity, RSD of retention time, Std/Blk intensity ratio, jaggedness, asymmetry factor, FWHM, and modality
- CSV metrics file with a validation column flagging metabolites passing the user-specified thresholds

## How to apply

For each chemical standard, extract its EIC from the input mzXML/mzML file using the specified m/z window, then apply IRLS (Iteratively Reweighted Least Squares) baseline correction followed by Savitzky-Golay smoothing on the EIC. From the smoothed profile detect the peak region and compute the maximum intensity and mass chromatographic quality (MCQ) index, then derive the asymmetry factor, FWHM, and jaggedness. Compute the modality metric, and when three replicates and blank files are supplied, compute the RSD of retention time across replicates and the Std/Blk intensity ratio. Aggregate all eight metrics per analyte into a metrics table written to CSV; peaks are validated only when Std/Blk exceeds a user-specified threshold r, RSD of retention time across the three replicates is below threshold c, and peak intensity exceeds threshold I.

## Related tools

- **MetaMOPE** (Containerized web tool and R source-code functions that generate the eight per-peak metrics (MCQ, maximum intensity, retention-time RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) from EICs) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions MetaMOPE provides for computing the multiple peak quality metrics) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container for MetaMOPE, launched via docker-compose so the peak-evaluation web service runs locally) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && cp .env.example .env && docker-compose up --build; then submit mzXML/mzML and blank files at http://localhost:13002
```

## Evaluation signals

- Each analyte row in the output CSV carries exactly the eight named metrics: MCQ, maximum intensity, retention-time RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, and modality
- A validation column marks each metabolite as validated only when Std/Blk > r, retention-time RSD across three replicates < c, and peak intensity > I
- MCQ, asymmetry factor, and maximum intensity are computed on the IRLS baseline-corrected, Savitzky-Golay-smoothed EIC, not the raw trace
- Retention-time RSD is only populated when the standard mixture was evaluated in three replicate LC/MS runs; Std/Blk only when a blank file is supplied
- Replicate processing yields a single consensus retention time per reference analyte derived from the three replicate EICs

## Limitations

- Validation of metabolites depends on user-specified thresholds (r, c, I); no universal defaults are stated in the article
- Metrics requiring replicates or blanks (retention-time RSD, Std/Blk) cannot be computed if those files are not supplied
- No changelog, discussion of limitations, or reproducibility gaps were available in the document's discussion section
- The peak quality metrics were validated in the context of a HILIC LC-MS platform and chemical standards; applicability to other chromatographic regimes is not established by this study

## Evidence

- [abstract] The extracted ion chromatograms of each chemical standard were retrieved (Supplementary Fig. S2), followed by baseline correction and smoothing of EIC using the IRLS algorithm and Savitzky-Golay: "The extracted ion chromatograms of each chemical standard were retrieved (Supplementary Fig. S2), followed by baseline correction and smoothing of EIC using the IRLS algorithm and Savitzky-Golay"
- [abstract] MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity: "MetaMOPE generated the following eight metrics for each peak: (i) MCQ, (ii) the maximum intensity, (iii) the RSD of retention time, (iv) the ratio of standard over blank (Std/Blk) intensity"
- [abstract] Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
- [abstract] In addition, MetaMOPE provides source codes of functions for multiple metrics written in R.: "In addition, MetaMOPE provides source codes of functions for multiple metrics written in R."
- [readme] Live version is freely available at https://metamope.cmdm.tw: "Live version is freely available at https://metamope.cmdm.tw"
