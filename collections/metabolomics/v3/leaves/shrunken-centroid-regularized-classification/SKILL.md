---
name: shrunken-centroid-regularized-classification
description: Use when peak-picked MSI pixel data (e.g. imzML-derived peaks, such as
  the pig fetus dataset with 10,200 mass features and 298 detected peaks) must be
  segmented into homogeneous tissue regions or classified into predefined classes.
when_to_use_negative:
- The pixels already have trusted class labels and the goal is pure prediction without
  feature selection — supervised image classification variants or other classifiers
  may be more appropriate
- 'The data are raw, un-peak-picked spectra: pre-processing (normalize, smooth, reduceBaseline,
  peakPick/peakAlign via process()) should be applied first'
- The dataset cannot be reduced to peak features and needs full-spectrum unsupervised
  exploration such as PCA, which Cardinal also provides
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3521
tools:
- name: Cardinal
  role: Provides spatialShrunkenCentroids() for regularized shrunken-centroid segmentation/classification
    of MSI pixels with the Spatially-Aware gaussian distance
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Execution environment in which Cardinal's segmentation and classification
    methods are run
- name: matter
  role: Cardinal's larger-than-memory backend that supports computing on big MSI datasets
    during segmentation
- name: BiocParallel
  role: Parallel processing support used alongside Cardinal processing steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:6495ca2e5d392a01ff7190192baa572cf3c7b434c32e73b4a3e03817fc3546b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# shrunken-centroid-regularized-classification

## Summary

Assigns pixels in mass spectrometry imaging (MSI) data to classes or tissue segments using nearest-shrunken-centroid mixture modeling with regularization, as implemented in Cardinal's spatialShrunkenCentroids() with the Spatially-Aware (gaussian) distance. It simultaneously selects informative mass features and characterizes assignment uncertainty.

## When to use

Use when peak-picked MSI pixel data (e.g. imzML-derived peaks, such as the pig fetus dataset with 10,200 mass features and 298 detected peaks) must be segmented into homogeneous tissue regions or classified into predefined classes.

## When NOT to use

- The pixels already have trusted class labels and the goal is pure prediction without feature selection — supervised image classification variants or other classifiers may be more appropriate
- The data are raw, un-peak-picked spectra: pre-processing (normalize, smooth, reduceBaseline, peakPick/peakAlign via process()) should be applied first
- The dataset cannot be reduced to peak features and needs full-spectrum unsupervised exploration such as PCA, which Cardinal also provides

## Inputs

- Peak-picked mass spectrometry imaging data loaded as a processed MSImagingExperiment (imzML or Analyze 7.5 via readMSIData())
- Matrix of peak-picked intensities per pixel (e.g. 28,016 non-background pixels x 10,200 mass features)
- For classification: predefined class labels for a training set of locations

## Outputs

- Per-pixel class/segment labels (spatial map of tissue segments)
- Number of selected segments (e.g. 11 tissue segments for the pig fetus data)
- Subset of informative ions (mass features) driving the segmentation/classification
- Summary statistics and runtime (e.g. segmentation_summary.csv; 241-827 s reported)

## How to apply

Load the peak-picked MSI data into Cardinal as a processed MSImagingExperiment (imzML can be read via readMSIData()). Run spatialShrunkenCentroids() on the peak-picked intensities with method='gaussian' to use the Spatially-Aware distance, scanning the regularization parameter to explore candidate solutions. Choose the regularization level that yields the desired segmentation (in the pig fetus case, 11 tissue segments, reported with runtimes of 241-827 s). For classification tasks with known labels, the same shrunken-centroid machinery assigns locations to pre-defined classes and estimates classification error by (cross-) validation. Record per-pixel labels, segment/class counts, and runtime, and export summary statistics (e.g. segmentation_summary.csv).

## Related tools

- **Cardinal** (Provides spatialShrunkenCentroids() for regularized shrunken-centroid segmentation/classification of MSI pixels with the Spatially-Aware gaussian distance) — github.com/kuwisdelu/Cardinal
- **R** (Execution environment in which Cardinal's segmentation and classification methods are run)
- **matter** (Cardinal's larger-than-memory backend that supports computing on big MSI datasets during segmentation)
- **BiocParallel** (Parallel processing support used alongside Cardinal processing steps)

## Examples

```
ssc <- spatialShrunkenCentroids(msi_peaks, method="gaussian", r=1, s=seq(0,30,5))
```

## Evaluation signals

- The chosen regularization solution yields the expected segment count (11 tissue segments for the pig fetus Figure 1b reproduction)
- Per-pixel labels are exported and cover all pixels (e.g. 28,016 non-background pixels) with no unassigned locations
- Runtime falls in the reported order of magnitude (241-827 s range for this dataset)
- The informative-ion subset selected by the model is non-empty and its m/z values are retained in the summary statistics
- Input object is a processed MSImagingExperiment built from peak-picked intensities, not raw spectra

## Limitations

- Statistical methods are based on mixture modeling and regularization, so the number of segments depends on the scanned regularization parameter and must be selected by the analyst
- Cardinal 3 dropped legacy support, so workflows written for older Cardinal versions may need revision
- Larger-than-memory handling relies on the rewritten matter backend; the raw-spectra-fit-in-memory assumption no longer holds for many modern MSI experiments

## Evidence

- [readme] The package includes classification and clustering methods based on nearest shrunken centroids, as well as traditional tools like PCA and PLS.: [evidence span withheld — anchor status: external_doc]
- [abstract] image classification, which assigns locations on the tissue to pre-defined classes, selects the subset of informative ions, and estimates the resulting classification error by (cross-) validation.: "image classification, which assigns locations on the tissue to pre-defined classes, selects the subset of informative ions, and estimates the resulting classification error by (cross-) validation."
- [abstract] image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions, and characterizes the associated uncertainty.: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions, and characterizes the associated"
- [abstract] The statistical methods are based on mixture modeling and regularization.: "The statistical methods are based on mixture modeling and regularization."
- [intro] Cardinal natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats via the readMSIData() and writeMSIData() functions.: "*Cardinal* natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats via the `readMSIData()` and `writeMSIData()` functions."
