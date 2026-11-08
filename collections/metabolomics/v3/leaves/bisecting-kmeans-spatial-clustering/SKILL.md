---
name: bisecting-kmeans-spatial-clustering
description: Use when a cluster of peaks with similar m/z signals has S1 and S2 similarity
  scores below user thresholds and needs its matrix-related ions separated from co-clustered
  isobaric or overlapping ions by their spatial distributions.
when_to_use_negative:
- Clusters whose S1 and S2 scores already exceed the user-specified thresholds — the
  paper states the overlapping detection algorithm is only executed in those clusters
  that report scores under a threshold
- Situations with no spatial ion-image information available, since the bisection
  is driven entirely by Pearson correlations between spatial distributions of ions
- When a simple, non-recursive peak-level matrix annotation without overlapping/isobaric
  disambiguation is sufficient (the recursive splitting is specifically for preventing
  misclassification of overlapped ions)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIcleanup
  role: Implements the bisecting k-means overlapping peak detection as part of the
    annotate_matrix workflow
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSI
  role: Loads the processed MSI dataset (ion images) required for computing spatial-distribution
    correlations
  repo: https://github.com/prafols/rMSI
- name: rMSIproc
  role: Loads the centroid peak matrix whose clusters are the input to the bisection
  repo: https://github.com/prafols/rMSIproc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:7931bb3ea6c164e65bbeffe23e37d44f5f42675096c04d7505c2322531c7dcdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bisecting-kmeans-spatial-clustering

## Summary

Recursive bisection of an experimental peak cluster into spatially coherent subgroups via k-means on Pearson-correlated ion images, used by rMSIcleanup to separate matrix-related ions from co-clustered isobaric or overlapping endogenous ions.

## When to use

Use when a cluster of peaks with similar m/z signals has S1 and S2 similarity scores below user thresholds and needs its matrix-related ions separated from co-clustered isobaric or overlapping ions by their spatial distributions.

## When NOT to use

- Clusters whose S1 and S2 scores already exceed the user-specified thresholds — the paper states the overlapping detection algorithm is only executed in those clusters that report scores under a threshold
- Situations with no spatial ion-image information available, since the bisection is driven entirely by Pearson correlations between spatial distributions of ions
- When a simple, non-recursive peak-level matrix annotation without overlapping/isobaric disambiguation is sufficient (the recursive splitting is specifically for preventing misclassification of overlapped ions)

## Inputs

- Experimental cluster of centroid peaks from a peak matrix (e.g. rMSIproc .zip) whose S1 and S2 similarity scores fall below user thresholds
- Full processed MSI dataset with ion images per peak (e.g. rMSI .tar file such as Ag_Pancreas_TOF_2015_Dataset2_proc.tar)
- User-specified S1/S2 similarity thresholds

## Outputs

- Per-peak table with m/z, assigned label (matrix-related, matrix-related with overlapping, or not-matrix-related), and subgroup/cluster membership
- Final bisection tree of subgroups with their recomputed S1 and S2 scores

## How to apply

Run the overlapping detection algorithm only on clusters whose S1 (spectral similarity, Pearson correlation between mean spectra) and S2 (intra-cluster morphological similarity) scores fall below the user-specified threshold, as rMSIcleanup does. First compute the Pearson correlation matrix between the ion images of all peaks in the cluster. Bisect the cluster's ions into two subgroups using k-means on their spatial-distribution correlations (bisecting k-means operating principle). Recompute S1 and S2 for each subgroup: if both surpass the thresholds, tag all ions in that subgroup as matrix-related and the remaining ions as matrix-related with overlapping, then terminate. Otherwise, recursively split both subgroups with k-means and rescore, stopping when a subgroup qualifies or when the largest subgroup contains fewer than half the peaks of the original cluster, in which case all ions are tagged not-matrix-related (no overlapping peaks). This recursion is what prevents overlapped or isobaric ions from being misclassified as matrix-related.

## Related tools

- **rMSIcleanup** (Implements the bisecting k-means overlapping peak detection as part of the annotate_matrix workflow) — https://github.com/gbaquer/rMSIcleanup
- **rMSI** (Loads the processed MSI dataset (ion images) required for computing spatial-distribution correlations) — https://github.com/prafols/rMSI
- **rMSIproc** (Loads the centroid peak matrix whose clusters are the input to the bisection) — https://github.com/prafols/rMSIproc

## Examples

```
pks<-rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); full<-rMSI::LoadMsiData("Ag_Pancreas_TOF_2015_Dataset2_proc.tar"); results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)
```

## Evaluation signals

- The recursive bisection terminates for every low-scoring cluster without adding matrix-related positives for high-scoring (above-threshold) clusters
- Output per-peak table exists, contains all peaks from the input cluster, and every peak carries one of the three labels: matrix-related, matrix-related with overlapping, or not-matrix-related
- For clusters qualifying at an early recursion, the 'matrix-related with overlapping' label is assigned to ions in the sibling subgroup rather than to the qualifying subgroup
- When the largest subgroup drops below half the peaks of the original cluster, all ions in the cluster are tagged not-matrix-related (no overlapping peaks), reflecting the paper's stopping rule
- Spectral similarity score S1 achieved AUC 0.97 and intra-cluster morphological similarity S2 achieved AUC 0.91 as classifiers — recomputed subgroup scores should remain in this range for silver-cluster-related m/z signals in AgLDI MSI data

## Limitations

- The algorithm only runs on clusters with S1 and S2 scores below the user threshold; above-threshold clusters are annotated without the overlapping bisection, so their spatial correlations are never examined
- Stopping when the largest subgroup contains fewer than half the peaks of the original cluster means genuinely mixed clusters may end up entirely labeled not-matrix-related
- The S1·S2 combined score matched S1 alone (AUC 0.97), so the morphological score S2 (AUC 0.91) alone may not suffice to distinguish subgroups in difficult cases

## Evidence

- [methods] Overlapping detection runs only on low-scoring clusters: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [methods] Bisection is driven by spatial correlations between ion images: [evidence span withheld — anchor status: not_found]
- [intro] Purpose is to prevent misclassification of overlapped/isobaric ions: [evidence span withheld — anchor status: not_found]
- [readme] Tool roles and data loading for the skill: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
- [readme] Purpose and scope of the package implementing the skill: "The algorithm takes into account the chemical formula and the spatial distribution to determine which ions are matrix-related. The algorithm incorporates an overlapping peak detection feature to"
- [methods] Classifier performance on silver-cluster-related m/z signals: [evidence span withheld — anchor status: not_found]
