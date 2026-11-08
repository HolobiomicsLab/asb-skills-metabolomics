---
name: mrm-peak-alignment
description: Use when processing multiple MRM/SRM samples whose EIC peak tables show
  retention time shifts between runs, and peaks must be aligned or EICs grouped automatically
  before manual inspection.
when_to_use_negative:
- Data are DDA/DIA acquisitions rather than MRM/SRM chromatograms — this skill aligns
  EIC chromatograms from targeted MRM runs
- Only a single sample is available, so no between-sample RT drift can be estimated
- The goal is vendor-format conversion or compression (use AirdPro/Aird tooling) rather
  than retention time alignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM analysis platform implementing the retention time drift estimation
    algorithm based on chromatogram similarity, used for peak alignment and automatic
    EIC grouping
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: Converts vendor raw files to Aird format, including support for MRM/SRM acquisition,
    providing the chromatogram data consumed by MRMPro
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:d35f8f40fc5dfa30fe4eee93806447443a906ac07307ba48a5793864c24c4999
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mrm-peak-alignment

## Summary

Retention time drift estimation and peak alignment for MRM mass spectrometry data, based on cosine similarity between extracted-ion chromatograms (EICs). MRMPro uses these estimated drifts for peak alignment and automatic EIC grouping prior to manual calibration.

## When to use

Use when processing multiple MRM/SRM samples whose EIC peak tables show retention time shifts between runs, and peaks must be aligned or EICs grouped automatically before manual inspection.

## When NOT to use

- Data are DDA/DIA acquisitions rather than MRM/SRM chromatograms — this skill aligns EIC chromatograms from targeted MRM runs
- Only a single sample is available, so no between-sample RT drift can be estimated
- The goal is vendor-format conversion or compression (use AirdPro/Aird tooling) rather than retention time alignment

## Inputs

- Extracted-ion chromatogram (EIC) peak tables for all MRM samples (e.g. as a pandas DataFrame)
- Transition group definitions identifying which EICs belong together

## Outputs

- rt_drift_estimates.csv with columns sample, transition group, and estimated RT drift (one drift estimate per input sample per group)
- Aligned peaks and automatically grouped EICs downstream of the drift estimates

## How to apply

Load the EIC peak tables for all MRM samples and, for each transition group, select the reference EIC by taking the arg-max of the product of peak count and maximum peak height per sample. For each non-reference sample, run a coarse grid search over candidate RT shifts to maximize cosine similarity between the shifted chromatogram and the reference, then refine the optimum with gradient descent on the RT shift while cosine similarity keeps increasing. Collect the converged drift per sample and transition group into rt_drift_estimates.csv, then use the estimated retention time drifts for peak alignment and automatic EIC grouping. The paper reports no numeric threshold for drift quality, so accept each drift if its cosine similarity is no lower than its grid-search initialization.

## Related tools

- **MRMPro** (Web-based MRM analysis platform implementing the retention time drift estimation algorithm based on chromatogram similarity, used for peak alignment and automatic EIC grouping) — https://github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (Converts vendor raw files to Aird format, including support for MRM/SRM acquisition, providing the chromatogram data consumed by MRMPro) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
drifts = estimate_rt_drift(eic_tables, metric='cosine', coarse='grid_search', refine='gradient_descent'); drifts.to_csv('rt_drift_estimates.csv')
```

## Evaluation signals

- Output table contains exactly one drift estimate per input sample per transition group
- Each reported drift attains a cosine similarity no lower than its grid-search initialization (acceptance check, since the paper reports no numeric threshold for drift quality)
- Gradient descent refinement converged only while cosine similarity was increasing
- Estimated drifts, when applied, yield consistent peak alignment and automatic EIC grouping across samples

## Limitations

- MRMPro relies on classical algorithms rather than more advanced analytical algorithms; incorporation of advanced algorithms is only planned future work
- Peak extraction accuracy is not yet improved by this method; improvement is deferred to future work
- The paper reports no numeric threshold for drift quality, so quality assessment relies on relative similarity checks rather than an absolute cutoff

## Evidence

- [abstract] we proposed a retention time drift estimation algorithm based on similarity of chromatograms: "we proposed a retention time drift estimation algorithm based on similarity of chromatograms"
- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms: "Despite demonstrating higher quantification accuracy compared to Skyline, MRMPro is built upon systematic optimizations of traditional algorithms"
- [readme] Supporting MRM acquisition method: [evidence span withheld — anchor status: external_doc]
