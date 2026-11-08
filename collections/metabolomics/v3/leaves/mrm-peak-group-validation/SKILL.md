---
name: mrm-peak-group-validation
description: Use when processing MRM peak/EIC feature lists that require grouping
  of chromatograms with similar shapes across samples, e.g. before batch manual peak
  inspection or when validating that EICs of one peptide transition co-cluster and
  noise stays unclustered.
when_to_use_negative:
- Input is already a validated, curated peak group table rather than raw EIC feature
  lists — grouping would not add information.
- The goal is peak detection/picking itself; MRMPro's discussion notes peak extraction
  accuracy improvement is deferred to future work and the grouping step assumes peaks
  have already been picked.
- Data are DDA/DIA acquisition outputs rather than targeted MRM transitions; the workflow
  is specific to MRM EICs.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform whose DBSCAN-based automatic EIC grouping
    supports batch manual inspection; the skill reproduces this grouping behavior
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client used upstream to convert vendor MRM files to the Aird format that
    MRMPro consumes
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for reading chromatogram index data (e.g. chromatogramIndex
    for MRM acquisition mode) from Aird files
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:cd4d3344a9b88455adc59ebbdab19d79d5ab8c45990ad3464cbe174514a4b00b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mrm-peak-group-validation

## Summary

Reproduces MRMPro's DBSCAN-based automatic EIC grouping, which co-clusters extracted-ion chromatograms with similar shapes so that the same target transition across samples lands in one group while low-intensity noise EICs remain unassigned. This supports MRMPro's core goal of faster batch manual inspection/calibration of MRM peak-picking errors.

## When to use

Use when processing MRM peak/EIC feature lists that require grouping of chromatograms with similar shapes across samples, e.g. before batch manual peak inspection or when validating that EICs of one peptide transition co-cluster and noise stays unclustered.

## When NOT to use

- Input is already a validated, curated peak group table rather than raw EIC feature lists — grouping would not add information.
- The goal is peak detection/picking itself; MRMPro's discussion notes peak extraction accuracy improvement is deferred to future work and the grouping step assumes peaks have already been picked.
- Data are DDA/DIA acquisition outputs rather than targeted MRM transitions; the workflow is specific to MRM EICs.

## Inputs

- Per-sample MRM EIC peak feature lists (retention times and intensities per transition)
- Retention-time drift estimates derived from chromatogram similarity
- Pairwise intensity differences between EICs

## Outputs

- EIC-to-cluster assignment table (e.g. eic_groups.csv)
- Grouping overview figure analogous to Fig. 7 (grouped signal EICs vs. separated noise)
- Noise/unclustered EIC labels (DBSCAN noise points)

## How to apply

Load per-sample EIC peak features (retention times and intensities) from the MRM dataset, and compute per-EIC retention-time drift values from the chromatogram-similarity-based drift estimates plus pairwise intensity differences between EICs. Min-max normalize the RT drifts to [-1,1] and the intensity differences to [0,1], then run DBSCAN (scikit-learn) on the normalized drift and intensity feature vectors, tuning eps/min_samples so that EICs with similar shapes co-cluster while low-intensity noise features are left unassigned. Write EIC-to-cluster assignments (e.g. eic_groups.csv) and render a grouping overview analogous to Fig. 7, showing grouped signal EICs versus separated noise. The rationale is MRMPro's stated objective: an automated peak grouping algorithm to better support batch manual calibration, since no peak-picking algorithm is perfect and manual inspection is still needed to correct errors.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform whose DBSCAN-based automatic EIC grouping supports batch manual inspection; the skill reproduces this grouping behavior) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client used upstream to convert vendor MRM files to the Aird format that MRMPro consumes) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for reading chromatogram index data (e.g. chromatogramIndex for MRM acquisition mode) from Aird files) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
from sklearn.cluster import DBSCAN; labels = DBSCAN(eps=e, min_samples=m).fit_predict(normalized_drift_and_intensity_features)  # assign clusters to eic_groups.csv
```

## Evaluation signals

- EICs of the same target peptide transition across samples fall into one DBSCAN cluster
- Low-intensity noise EICs remain unassigned (DBSCAN noise label), matching the Fig. 7 separation of grouped signal vs. noise
- Normalized RT drift values lie in [-1,1] and normalized intensity differences in [0,1] before clustering (min-max scaling check)
- Grouping overview figure shows EICs with similar shapes co-clustered, consistent with Fig. 7
- Downstream: grouped EICs enable faster batch manual calibration — no peak-picking errors left unflagged that would require manual grouping

## Limitations

- MRMPro is built on systematic optimizations of traditional/classical algorithms; incorporation of more advanced analytical algorithms is only planned future work.
- Peak extraction accuracy itself is not yet improved; the grouping supports manual inspection but does not fix peak picking.
- As the sample queue grows, the cost of manual inspection (which grouping only accelerates) becomes higher and higher.
- DBSCAN requires tuning eps/min_samples; poor settings may merge noise with signal EICs or over-fragment groups.

## Evidence

- [other] MRMPro's EIC grouping algorithm groups EICs via DBSCAN clustering results, assigning EICs with similar shapes to the same group as illustrated in Fig. 7.: [evidence span withheld — anchor status: not_found]
- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] An automated peak grouping algorithm to better support batch manual calibration: "An automated peak grouping algorithm to better support batch manual calibration"
- [abstract] no peak picking algorithm can achieve perfect accuracy, manual inspection is necessary to correct the errors: "no peak picking algorithm can achieve perfect accuracy, manual inspection is necessary to correct the errors"
- [readme] Chromatogram information for MRM acquisition mode: [evidence span withheld — anchor status: external_doc]
