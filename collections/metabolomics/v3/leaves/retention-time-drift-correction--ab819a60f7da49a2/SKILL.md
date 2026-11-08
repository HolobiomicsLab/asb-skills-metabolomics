---
name: retention-time-drift-correction
description: Use when MRM chromatographic peak lists from multiple samples show retention
  time shifts that must be corrected before peak alignment or EIC grouping, and per-sample
  RT drift values need to be estimated from chromatogram shapes.
when_to_use_negative:
- Input data is not MRM chromatographic (EIC) data — the algorithm assumes per-sample
  EIC peak lists with retention times and intensities
- Peaks are already aligned or drift values are already known (e.g. from internal
  standards), making chromatogram-similarity drift estimation redundant
- The goal is raw file format conversion or compression rather than drift estimation
  (use AirdPro/Aird SDK instead)
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based platform implementing the chromatogram-similarity-based RT drift
    estimation, whose drift values are then used for peak alignment and automatic
    EIC grouping
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: Converts vendor MRM raw files into the Aird format that MRMPro reads for fast
    chromatogram rendering
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK for reading Aird-format MRM chromatogram data feeding downstream
    drift estimation
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:0570d4f75dfb0ec043521f34f349b99086448a7394d417a2d740346b30fa7e10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retention-time-drift-correction

## Summary

Estimates retention time (RT) drifts between MRM samples by aligning each extracted-ion chromatogram (EIC) to a reference EIC using combined grid search and gradient descent to maximize cosine similarity. The resulting drift estimates feed MRMPro's peak alignment and automatic EIC grouping.

## When to use

Use when MRM chromatographic peak lists from multiple samples show retention time shifts that must be corrected before peak alignment or EIC grouping, and per-sample RT drift values need to be estimated from chromatogram shapes.

## When NOT to use

- Input data is not MRM chromatographic (EIC) data — the algorithm assumes per-sample EIC peak lists with retention times and intensities
- Peaks are already aligned or drift values are already known (e.g. from internal standards), making chromatogram-similarity drift estimation redundant
- The goal is raw file format conversion or compression rather than drift estimation (use AirdPro/Aird SDK instead)

## Inputs

- Extracted-ion chromatogram (EIC) peak tables for all MRM samples (retention time, intensity, FWHM per peak), e.g. as a pandas DataFrame
- MRM transition group identifiers linking EICs across samples

## Outputs

- rt_drift_estimates.csv: one drift estimate per input sample per transition group (columns: sample, transition group, estimated RT drift)
- Per-sample RT drift values consumed downstream by peak alignment and DBSCAN-based EIC grouping

## How to apply

Load the extracted-ion chromatogram (EIC) peak tables for all MRM samples into a pandas DataFrame. For each transition group, select the reference EIC by computing the product of peak count and maximum peak height per sample and taking the arg-max sample as reference. For each non-reference sample, run a coarse grid search over candidate RT shifts to maximize the cosine similarity between the shifted chromatogram and the reference, then refine each grid-search optimum with gradient descent on the RT shift parameter, continuing while cosine similarity increases. Collect the converged drift value for every sample into a table with columns sample, transition group, and estimated RT drift (rt_drift_estimates.csv); these drifts are subsequently consumed by MRMPro's RT-offset-based peak alignment (iterative FWHM-bounded grouping) and the DBSCAN-based automatic EIC grouping, where drift values are min-max normalized to [-1,1].

## Related tools

- **MRMPro** (Web-based platform implementing the chromatogram-similarity-based RT drift estimation, whose drift values are then used for peak alignment and automatic EIC grouping) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (Converts vendor MRM raw files into the Aird format that MRMPro reads for fast chromatogram rendering) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK for reading Aird-format MRM chromatogram data feeding downstream drift estimation) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
import pandas as pd; eics = pd.read_csv('eic_peaks.csv'); ref = eics.groupby('sample').apply(lambda d: (d.peak_count*d.max_height).idxmax()); # grid search + gradient descent per sample to maximize cosine similarity vs reference, then write rt_drift_estimates.csv
```

## Evaluation signals

- rt_drift_estimates.csv contains exactly one drift estimate per input sample per transition group
- Each reported drift attains a cosine similarity no lower than its grid-search initialization (paper reports no numeric threshold for drift quality)
- Every non-reference sample has a converged drift value while the reference sample has drift 0 by construction
- Downstream invariant: after drift adjustment, every aligned peak lies within the FWHM window of its alignment group's reference RT
- In EIC grouping, EICs of the same target peptide transition across samples co-cluster after drift normalization to [-1,1], and noise EICs remain unclustered by DBSCAN

## Limitations

- The paper reports no numeric threshold for acceptable drift quality; only relative improvement over the grid-search initialization is verifiable
- MRMPro relies on classical algorithms (grid search + gradient descent) rather than more advanced analytical algorithms; incorporation of advanced algorithms to improve peak extraction accuracy is only planned future work
- A reference EIC must be identifiable per transition group; the reference is chosen by the product of peak count and maximum peak height, which can be sensitive to low-quality samples

## Evidence

- [methods] Chromatogram-similarity reference selection: [evidence span withheld — anchor status: not_found]
- [methods] Combined grid search and gradient descent: [evidence span withheld — anchor status: not_found]
- [abstract] Drifts used downstream: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [discussion] Classical algorithms limitation: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] Aird SDK supports MRM chromatograms: [evidence span withheld — anchor status: external_doc]
