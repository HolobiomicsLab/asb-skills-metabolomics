---
name: mass-spectral-peak-overlap-detection
description: Use when a matrix-annotation score (e.g. spectral S1 or morphological
  S2 in rMSIcleanup) falls below the user threshold and overlapped or isobaric peaks
  may share the same ion image as matrix-related ions, risking misclassification.
when_to_use_negative:
- Clusters whose S1 and S2 scores are already above the user-specified threshold —
  the paper states the overlapping detection algorithm is only executed for below-threshold
  clusters
- Data that has not been pre-processed into a centroid-mode peak matrix with rMSIproc
  (the module operates on peak matrices and ion images, not raw profile-mode imzML)
- Annotation problems where matrix chemical composition is unknown — matrix-related
  ions are determined from the matrix chemical formula and spatial distribution, which
  this module depends on
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2640
tools:
- name: rMSIcleanup
  role: Hosts the overlapping peak detection feature; supplies the S1/S2-scored clusters
    that trigger the bisecting k-means splitting and provides annotate_matrix for
    the enclosing workflow
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Loads the centroid-mode peak matrix (.zip) that contains the peaks to be classified;
    also stores the cleaned peak matrix after annotation
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed MSI data (.tar) supplying the ion images whose Pearson
    correlations drive the k-means bisection
- name: R
  role: Implementation language for the recursive k-means bisection, Pearson correlation
    computation, and S1/S2 rescoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:4bb2be1012f78f8bd084d891da5e9d1e62b47519c18e3cd4e61d13a48ac06ee3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectral-peak-overlap-detection

## Summary

A bisecting k-means-based overlapping peak detection module that flags overlapped or isobaric ions inside low-scoring rMSIcleanup clusters so they are not misclassified as matrix-related peaks. It recursively splits clusters by spatial-distribution correlation and rescoring until subgroups qualify or become too small.

## When to use

Use when a matrix-annotation score (e.g. spectral S1 or morphological S2 in rMSIcleanup) falls below the user threshold and overlapped or isobaric peaks may share the same ion image as matrix-related ions, risking misclassification.

## When NOT to use

- Clusters whose S1 and S2 scores are already above the user-specified threshold — the paper states the overlapping detection algorithm is only executed for below-threshold clusters
- Data that has not been pre-processed into a centroid-mode peak matrix with rMSIproc (the module operates on peak matrices and ion images, not raw profile-mode imzML)
- Annotation problems where matrix chemical composition is unknown — matrix-related ions are determined from the matrix chemical formula and spatial distribution, which this module depends on

## Inputs

- Peak matrix in centroid mode (.zip) readable by rMSIproc::LoadPeakMatrix
- Processed MSI dataset in rMSI format (.tar) readable by rMSI::LoadMsiData
- User-specified S1 and S2 similarity-score thresholds
- Experimental clusters with computed S1/S2 scores from the rMSIcleanup annotation step

## Outputs

- Per-peak table with m/z, assigned label (matrix-related, matrix-related with overlapping, or not-matrix-related), and subgroup/cluster membership (overlapping_peak_flags.csv)
- Updated cluster/subgroup S1 and S2 similarity scores

## How to apply

Load the sample peak matrix with rMSIproc::LoadPeakMatrix and the processed spectra with rMSI::LoadMsiData (e.g. Ag_Pancreas_TOF_2015_Dataset2 from Mendeley doi 10.17632/vsk68tjcgh.1). For every experimental cluster whose S1 (spectral similarity) and S2 (intra-cluster morphological similarity) scores fall under the user-specified threshold, compute the Pearson correlation matrix between the ion images of the cluster's peaks. Bisect the cluster's ions into two subgroups with k-means on their spatial-distribution correlations, then recompute S1 and S2 for each subgroup: if both exceed the thresholds, tag all ions in the subgroup as matrix-related and the remaining ions as matrix-related with overlapping. Otherwise recursively split both subgroups, stopping when a subgroup qualifies or when the largest subgroup contains fewer than half the peaks of the original cluster, in which case all ions are tagged not-matrix-related (no overlapping peaks). The rationale is that overlapping detection is only executed for below-threshold clusters, so above-threshold (negative-class) scores remain unchanged, mirroring the paper's observation.

## Related tools

- **rMSIcleanup** (Hosts the overlapping peak detection feature; supplies the S1/S2-scored clusters that trigger the bisecting k-means splitting and provides annotate_matrix for the enclosing workflow) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Loads the centroid-mode peak matrix (.zip) that contains the peaks to be classified; also stores the cleaned peak matrix after annotation) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed MSI data (.tar) supplying the ion images whose Pearson correlations drive the k-means bisection)
- **R** (Implementation language for the recursive k-means bisection, Pearson correlation computation, and S1/S2 rescoring)

## Examples

```
pks<-rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); full<-rMSI::LoadMsiData("Ag_Pancreas_TOF_2015_Dataset2_proc.tar"); results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)
```

## Evaluation signals

- The output CSV exists and contains all three classification labels (matrix-related, matrix-related with overlapping, not-matrix-related) for every peak
- The recursive bisection terminates for every low-scoring cluster, never exceeding the stopping rule (largest subgroup < half the peaks of the original cluster)
- Above-threshold (high-scoring) clusters receive no new positives — overlapping detection leaves negative-class scores unchanged, matching the paper's observation
- Each flagged 'matrix-related with overlapping' peak's subgroup achieves both S1 and S2 scores above the user thresholds after splitting
- m/z values in the output table match the peak matrix loaded from Ag_Pancreas_TOF_2015_Dataset2

## Limitations

- The algorithm is only triggered for clusters below the user-specified S1/S2 threshold, so overlapping detection cannot correct misclassifications in clusters already scored above threshold
- Bisection halts and tags all ions not-matrix-related once the largest subgroup falls below half the original cluster's peak count, which may leave genuinely overlapping ions unflagged in very small clusters
- Validation is grounded in 14 AgLDI datasets exploiting silver's characteristic isotopic pattern; applicability to other LDI-promoting materials relies on the same formula-and-spatial-distribution logic
- rMSIcleanup depends on rMSI (ref 0.8) and rMSIproc (ref 0.2) fixed release tags; other versions are untested

## Evidence

- [methods] Overlapping peak detection is only run on below-threshold clusters: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [readme] The algorithm incorporates overlapping peak detection to avoid misclassification of overlapped/isobaric ions: "The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions."
- [readme] Load peak matrix and processed data with rMSIproc and rMSI: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
- [methods] S1 and S2 classifier performance: [evidence span withheld — anchor status: not_found]
- [abstract] Matrix-related ions are determined from formula and spatial distribution: "We have developed rMSIcleanup, an open-source R package to annotate and remove matrix-related signals based on its chemical formula and the spatial distribution of its ions."
