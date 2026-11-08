---
name: ion-image-correlation-analysis
description: Use when an rMSIproc peak matrix has matrix-related clusters (e.g. silver
  'Ag1' annotation) whose S1 spectral and S2 morphological similarity scores fall
  below the user-specified threshold, so co-localized endogenous ions may be hiding
  inside the cluster.
when_to_use_negative:
- The cluster's S1 and S2 similarity scores already exceed the user-specified threshold
  — the paper states the overlapping detection algorithm is only executed for under-threshold
  clusters.
- Input is raw profile-mode imzML that has not been converted to the rMSIproc peak-matrix
  format (centroid mode, SNR 5, peaks in <5% of pixels filtered, 6-point binning).
- The goal is matrix annotation by blank/reference-area subtraction — the article
  showed blank subtraction reached at most AUC 0.61 versus rMSIcleanup's 0.97 and
  cannot handle overlapped/isobaric peaks.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rMSIcleanup
  role: 'Implements the overlapping peak detection: Pearson correlation of ion images
    plus bisecting k-means splitting and rescoring of under-threshold matrix clusters'
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Loads the centroid-mode peak matrix (.zip) that supplies the peaks and ion
    images to be classified
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed (.tar) MSI dataset providing the full spectra and ion
    images used in correlation and scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:33e84c0ad3dfd11aa860ec9c1a368add9131fc9ceb3c79f1719c155bd18f465a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# ion-image-correlation-analysis

## Summary

Overlapping peak detection from the rMSIcleanup package: uses Pearson correlation between ion images and bisecting k-means to split matrix-related clusters, preventing overlapped or isobaric endogenous ions from being misclassified as matrix-related.

## When to use

Use when an rMSIproc peak matrix has matrix-related clusters (e.g. silver 'Ag1' annotation) whose S1 spectral and S2 morphological similarity scores fall below the user-specified threshold, so co-localized endogenous ions may be hiding inside the cluster.

## When NOT to use

- The cluster's S1 and S2 similarity scores already exceed the user-specified threshold — the paper states the overlapping detection algorithm is only executed for under-threshold clusters.
- Input is raw profile-mode imzML that has not been converted to the rMSIproc peak-matrix format (centroid mode, SNR 5, peaks in <5% of pixels filtered, 6-point binning).
- The goal is matrix annotation by blank/reference-area subtraction — the article showed blank subtraction reached at most AUC 0.61 versus rMSIcleanup's 0.97 and cannot handle overlapped/isobaric peaks.

## Inputs

- rMSIproc peak matrix (.zip), e.g. Ag_Pancreas_TOF_2015_Dataset2.zip
- Processed MSI dataset in rMSI format (.tar), e.g. Ag_Pancreas_TOF_2015_Dataset2_proc.tar
- Matrix chemical formula (e.g. "Ag1") for the LDI-promoting material
- User-specified S1 (spectral) and S2 (morphological) similarity-score thresholds

## Outputs

- Per-peak annotation table with m/z, assigned label (matrix-related / matrix-related with overlapping / not-matrix-related), and subgroup/cluster membership
- overlapping_peak_flags.csv
- Visual PDF report justifying each matrix annotation
- Cleaned peak matrix (after remove_matrix) storable via rMSIproc::StorePeakMatrix

## How to apply

Load the peak matrix with rMSIproc::LoadPeakMatrix and the processed spectra with rMSI::LoadMsiData, then run rMSIcleanup::annotate_matrix with the matrix formula (e.g. "Ag1"). For each cluster whose S1 (spectral) and S2 (intra-cluster morphological) scores are below the user-specified threshold — the paper reports S1 as the best classifier with AUC 0.97 and S2 with AUC 0.91 — compute the Pearson correlation matrix between the ion images of the cluster's peaks. Apply bisecting k-means on these spatial-distribution correlations to split the cluster's ions into two subgroups, then recompute S1 and S2 for each subgroup. If both scores surpass the thresholds, tag all ions in the subgroup as matrix-related and the remaining ions as matrix-related with overlapping; otherwise recursively split both subgroups. Stop when the largest subgroup contains fewer than half the peaks of the original cluster, in which case all ions are labelled not-matrix-related (no overlapping peaks).

## Related tools

- **rMSIcleanup** (Implements the overlapping peak detection: Pearson correlation of ion images plus bisecting k-means splitting and rescoring of under-threshold matrix clusters) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Loads the centroid-mode peak matrix (.zip) that supplies the peaks and ion images to be classified) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed (.tar) MSI dataset providing the full spectra and ion images used in correlation and scoring)

## Examples

```
pks<-rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); full<-rMSI::LoadMsiData("Ag_Pancreas_TOF_2015_Dataset2_proc.tar"); results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)
```

## Evaluation signals

- overlapping_peak_flags.csv exists and contains one of the three classification labels for every peak in the matrix
- The recursive bisection terminates for every cluster whose S1 and S2 scores fall below the user threshold
- No overlapping positives are added for high-scoring (above-threshold) clusters, mirroring the paper's observation that overlapping detection leaves negative-class scores unchanged
- On silver-assisted validation datasets, S1 achieves AUC ≈ 0.97 and S2 ≈ 0.91 as classifiers of matrix-related peaks
- The generated PDF visual report exists and justifies each annotation

## Limitations

- Overlapping detection is only executed on clusters reporting S1 and S2 scores under the user-specified threshold; above-threshold clusters are never examined.
- If recursion stops because the largest subgroup contains fewer than half the peaks of the original cluster, all ions in that cluster are tagged not-matrix-related — a possible source of false negatives.
- The method was validated on 14 Ag-sputtered (silver-assisted LDI) MSI datasets from three laboratories, exploiting the characteristic silver isotopic pattern; performance on other matrix/substrate combinations is not established here.
- Downstream of this step, matrix removal assumes the annotation is correct; mislabeled overlapping ions would be removed along with true matrix peaks.

## Evidence

- [methods] Overlapping detection runs only for under-threshold clusters: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [readme] README states purpose of overlapping peak detection: "The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions."
- [intro] Algorithm rationale relies on ion images plus chemical information: "we propose a new algorithm that relies not only on the ion images but also on the chemical information of the LDI promoting material used."
- [results] Classifier performance of S1 and S2: [evidence span withheld — anchor status: not_found]
