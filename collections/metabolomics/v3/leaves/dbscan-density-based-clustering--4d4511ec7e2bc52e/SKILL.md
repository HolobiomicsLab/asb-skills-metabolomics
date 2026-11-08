---
name: dbscan-density-based-clustering
description: Use when MRM/EIC peak features must be partitioned into groups of chromatograms
  with similar shapes — e.g., grouping same-transition EICs across samples while leaving
  low-intensity noise unclustered — as a step toward automated batch manual calibration.
when_to_use_negative:
- Input is already grouped or aligned — clustering again would duplicate MRMPro's
  peak alignment and EIC grouping stage
- Data are not MRM chromatogram/EIC features (e.g., untargeted DDA/DIA spectra or
  feature tables from other modalities), where shape-based EIC grouping does not apply
- The goal is peak detection or quantification accuracy — the paper notes MRMPro does
  not yet improve peak extraction accuracy, only manual inspection efficiency
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: Web-based MRM analysis platform implementing the automatic EIC grouping algorithm
    that assigns EICs with similar shapes to the same group based on DBSCAN clustering
    results
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: scikit-learn
  role: DBSCAN implementation used on normalized retention-time-drift and intensity-difference
    feature vectors
- name: Aird-SDK
  role: Provides programmatic access to chromatogram data (including MRM chromatogramIndex)
    stored in the Aird format, which can feed the EIC features used for grouping
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:a7c646af887f7f043e5dadcdbc0977a45d0009ea86782eca36860f1a8560adf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - Aird-SDK
  - MRMPro
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dbscan-density-based-clustering

## Summary

DBSCAN-based density clustering used by MRMPro to automatically group extracted-ion chromatograms (EICs) from MRM data, so that EICs with similar peak shapes co-cluster and support batch manual calibration instead of per-sample manual inspection.

## When to use

Use when MRM/EIC peak features must be partitioned into groups of chromatograms with similar shapes — e.g., grouping same-transition EICs across samples while leaving low-intensity noise unclustered — as a step toward automated batch manual calibration.

## When NOT to use

- Input is already grouped or aligned — clustering again would duplicate MRMPro's peak alignment and EIC grouping stage
- Data are not MRM chromatogram/EIC features (e.g., untargeted DDA/DIA spectra or feature tables from other modalities), where shape-based EIC grouping does not apply
- The goal is peak detection or quantification accuracy — the paper notes MRMPro does not yet improve peak extraction accuracy, only manual inspection efficiency

## Inputs

- Per-sample EIC peak lists from MRM acquisition (retention times and intensities per transition)
- Retention-time drift values estimated via chromatogram-similarity-based drift estimation
- Pairwise intensity differences between EICs

## Outputs

- EIC-to-cluster assignment table (e.g., eic_groups.csv)
- Grouping overview figure (analogous to Fig. 7) showing grouped signal EICs versus separated noise

## How to apply

Load per-sample EIC peak features (retention times and intensities) from the MRM dataset, then derive per-EIC retention-time drift values using MRMPro's chromatogram-similarity-based drift estimates and pairwise intensity differences between EICs. Min-max normalize the retention-time drift values to [-1, 1] and the intensity differences to [0, 1], because DBSCAN is distance-based and these two feature types have different natural scales. Run DBSCAN (scikit-learn) on the concatenated normalized drift/intensity feature vectors, tuning eps and min_samples so that EICs with similar shapes fall into the same cluster while low-intensity noise features remain unassigned (label -1). Write the EIC-to-cluster assignments (e.g., to eic_groups.csv) and render a grouping overview analogous to the paper's Fig. 7. Rationale: MRMPro deliberately relies on classical algorithms like DBSCAN to provide a stable, precise analytical framework for automating what would otherwise be slow manual grouping during batch manual inspection.

## Related tools

- **MRMPro** (Web-based MRM analysis platform implementing the automatic EIC grouping algorithm that assigns EICs with similar shapes to the same group based on DBSCAN clustering results) — https://github.com/CSi-Studio/MRMPro-Doc
- **scikit-learn** (DBSCAN implementation used on normalized retention-time-drift and intensity-difference feature vectors)
- **Aird-SDK** (Provides programmatic access to chromatogram data (including MRM chromatogramIndex) stored in the Aird format, which can feed the EIC features used for grouping) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
from sklearn.cluster import DBSCAN; eic_groups = DBSCAN(eps=eps_tuned, min_samples=min_samples_tuned).fit_predict(norm_rt_drift_and_intensity_features)
```

## Evaluation signals

- EICs of the same target peptide transition across samples fall into a single cluster, matching the grouping behavior illustrated in Fig. 7
- Low-intensity noise EICs remain unclustered (DBSCAN label -1) rather than being force-assigned to signal groups
- Retention-time drift features lie in [-1, 1] and intensity-difference features in [0, 1] after min-max normalization, with no NaNs or unscaled columns entering DBSCAN
- Every EIC in the input appears exactly once in the eic_groups.csv assignment table (no dropped or duplicated EICs)
- Rendered grouping overview separates signal EIC clusters from noise, consistent with MRMPro's automated peak grouping for batch manual calibration

## Limitations

- DBSCAN is a classical algorithm; the authors state that more advanced analytical algorithms are only planned future work for improving peak extraction accuracy and manual inspection efficiency
- eps and min_samples must be tuned per dataset; mis-tuned parameters can co-cluster noise with signal or split EICs of the same transition
- Feature quality depends on the upstream retention-time drift estimation based on chromatogram similarity; poor drift estimates degrade grouping

## Evidence

- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] An automated peak grouping algorithm to better support batch manual calibration: "An automated peak grouping algorithm to better support batch manual calibration"
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] ChromatogramIndex: Chromatogram information for MRM acquisition mode: [evidence span withheld — anchor status: external_doc]
