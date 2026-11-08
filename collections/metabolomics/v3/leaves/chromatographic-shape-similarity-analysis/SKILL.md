---
name: chromatographic-shape-similarity-analysis
description: Use when processing MRM acquisition data where EIC peak features across
  samples must be grouped by chromatographic shape (e.g., same target peptide transition
  across samples), and manual inspection needs pre-grouped EICs to keep low-intensity
  noise features separated from real signal.
when_to_use_negative:
- Data are DDA/DIA acquisitions rather than MRM — MRMPro's EIC grouping and collaborative
  manual inspection are dedicated to MRM data analysis in large volumes
- The goal is peak detection/picking accuracy itself — the article notes peak extraction
  accuracy is not yet improved and improvement is deferred to future work; the grouping
  only organizes already-picked EIC features
- A feature table with pre-assigned transition groups already exists — regrouping
  via DBSCAN would duplicate existing group information
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform providing the chromatogram-similarity-based
    retention time drift estimation and the automatic EIC grouping that feeds batch
    manual inspection
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client converting vendor MRM/SRM files (e.g., Wiff2) into the compressed
    Aird format consumed downstream
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Developer SDK (Java, C#, Python) for reading Aird MRM chromatogram data during
    secondary development
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: scikit-learn
  role: Provides the DBSCAN clustering used to group normalized EIC drift/intensity
    feature vectors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:4d36a13060c70a3443cc85a6c0eb26199a9ab09c491f96a95295d1410bb2030c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# chromatographic-shape-similarity-analysis

## Summary

Groups extracted-ion-chromatogram (EIC) peak features from MRM datasets by clustering EICs with similar chromatographic shapes, as implemented in MRMPro using DBSCAN on normalized retention-time drift and intensity-difference features. It automates EIC grouping so that batch manual calibration can proceed on coherent transition groups rather than per-EIC.

## When to use

Use when processing MRM acquisition data where EIC peak features across samples must be grouped by chromatographic shape (e.g., same target peptide transition across samples), and manual inspection needs pre-grouped EICs to keep low-intensity noise features separated from real signal.

## When NOT to use

- Data are DDA/DIA acquisitions rather than MRM — MRMPro's EIC grouping and collaborative manual inspection are dedicated to MRM data analysis in large volumes
- The goal is peak detection/picking accuracy itself — the article notes peak extraction accuracy is not yet improved and improvement is deferred to future work; the grouping only organizes already-picked EIC features
- A feature table with pre-assigned transition groups already exists — regrouping via DBSCAN would duplicate existing group information

## Inputs

- EIC peak feature lists from MRM datasets (per-sample peak lists with retention times and intensities)
- Aird-format MRM files (converted from vendor files such as Wiff2 by AirdPro, indexed via chromatogramIndex for MRM acquisition mode)
- Chromatogram-similarity-based retention time drift estimates per EIC

## Outputs

- EIC-to-cluster assignment table (e.g., eic_groups.csv)
- Grouping overview figure analogous to Fig. 7 showing grouped signal EICs versus separated noise EICs

## How to apply

Load per-sample EIC peak lists with retention times and intensities from the MRM dataset (Aird format converted from vendor files supports MRM/SRM acquisition via AirdPro, readable through AirdSDK). Compute per-EIC retention-time drift values across samples using MRMPro's chromatogram-similarity-based drift estimation, and derive pairwise intensity differences between EICs. Min-max normalize the retention-time drift values to [-1,1] and the intensity differences to [0,1], then run DBSCAN (scikit-learn) on the combined normalized feature vectors, tuning eps and min_samples so that EICs with similar shapes co-cluster while low-intensity noise features remain unassigned (label -1). Write EIC-to-cluster assignments (e.g., eic_groups.csv) and render a grouping overview analogous to Fig. 7 of the MRMPro paper. The rationale is that MRMPro, as the first tool focused on improving efficiency of batch manual inspection for MRM data, uses this automated peak grouping to better support batch manual calibration, relying on classical algorithms for a stable analytical framework.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform providing the chromatogram-similarity-based retention time drift estimation and the automatic EIC grouping that feeds batch manual inspection) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client converting vendor MRM/SRM files (e.g., Wiff2) into the compressed Aird format consumed downstream) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Developer SDK (Java, C#, Python) for reading Aird MRM chromatogram data during secondary development) — https://github.com/CSi-Studio/Aird-SDK
- **scikit-learn** (Provides the DBSCAN clustering used to group normalized EIC drift/intensity feature vectors)

## Examples

```
from sklearn.cluster import DBSCAN; labels = DBSCAN(eps=eps, min_samples=k).fit_predict(X_norm)
```

## Evaluation signals

- EICs of the same target peptide transition across samples fall into a single DBSCAN cluster
- Low-intensity noise EICs remain unclustered (DBSCAN label -1), consistent with the signal/noise separation illustrated in Fig. 7 of the MRMPro paper
- Normalized features respect their specified ranges: retention-time drift values within [-1,1] and intensity differences within [0,1]
- eic_groups.csv contains one row per EIC with a valid cluster assignment and no missing mappings
- The rendered grouping overview visually separates grouped signal EICs from separated noise, matching the behavior described in the source workflow

## Limitations

- MRMPro relies on classical algorithms (DBSCAN among them) rather than more advanced analytical algorithms; incorporation of advanced algorithms is only planned future work
- No peak picking algorithm achieves perfect accuracy, so manual inspection is still necessary; grouping improves but does not eliminate manual correction
- MRMPro is currently dedicated to improving the efficiency of manual inspection; broader analytical capability is not yet implemented
- eps/min_samples for DBSCAN require tuning; poorly chosen values can merge noise into signal groups or fragment true transition groups

## Evidence

- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] An automated peak grouping algorithm to better support batch manual calibration: "An automated peak grouping algorithm to better support batch manual calibration"
- [results] MRMPro's EIC grouping algorithm groups EICs via DBSCAN clustering results, assigning EICs with similar shapes to the same group as illustrated in Fig. 7.: [evidence span withheld — anchor status: not_found]
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] [New Feature] Supporting MRM acquisition method: [evidence span withheld — anchor status: external_doc]
