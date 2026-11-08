---
name: differential-peak-enrichment-analysis
description: Use when a fitted MSI segmentation exists (e.g., spatial shrunken centroids
  in Cardinal on imzML-derived pig fetus liver/heart data) and you must report which
  peaks each segment differentially enriches or lacks.
when_to_use_negative:
- When data have pre-defined class labels and the goal is supervised image classification
  with cross-validated error estimation rather than characterizing unsupervised segmentation
  segments.
- When the segmentation has not yet been fit — the t-statistics are only interpretable
  relative to an existing segmentation of the tissue into homogeneous chemical regions.
- When input spectra are raw and not yet peak-processed; run Cardinal pre-processing
  (normalize, smooth, reduceBaseline, peakPick/peakAlign) first.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cardinal
  role: R package providing the fitted segmentation and the topFeatures() t-statistic
    computation used to rank enriched and absent m/z peaks per segment
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment for loading the segmentation, running topFeatures(), ranking
    peaks, and writing CSV tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:0a666579dfe1cd884f9ad7f4a9d90364ed24d086d3e73f3037d124c503378860
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-peak-enrichment-analysis

## Summary

Quantifies which m/z peaks are systematically enriched or absent in a specific image segment (e.g., liver or heart in pig fetus MSI sections) by ranking per-segment t-statistics from a fitted spatial shrunken centroids segmentation, then validating against reported enriched/absent peak counts.

## When to use

Use when a fitted MSI segmentation exists (e.g., spatial shrunken centroids in Cardinal on imzML-derived pig fetus liver/heart data) and you must report which peaks each segment differentially enriches or lacks.

## When NOT to use

- When data have pre-defined class labels and the goal is supervised image classification with cross-validated error estimation rather than characterizing unsupervised segmentation segments.
- When the segmentation has not yet been fit — the t-statistics are only interpretable relative to an existing segmentation of the tissue into homogeneous chemical regions.
- When input spectra are raw and not yet peak-processed; run Cardinal pre-processing (normalize, smooth, reduceBaseline, peakPick/peakAlign) first.

## Inputs

- Fitted image segmentation result from spatial shrunken centroids on the pig fetus liver/heart MSI dataset (Cardinal object in R)
- Peak-picked m/z feature table from the processed mass spectra (after normalize, reduceBaseline, peakPick/peakAlign pre-processing)
- Segment membership labels for each pixel (liver segment, heart segment)

## Outputs

- Ranked CSV table of enriched peaks for the liver segment (positive t-statistics; expected 92 peaks)
- Ranked CSV table of absent peaks for the liver segment (negative t-statistics; expected 153 peaks)
- Ranked CSV tables of enriched (23 peaks) and absent (0 peaks) peaks for the heart segment

## How to apply

Load the fitted segmentation result in R with the Cardinal package, then apply topFeatures() to compute per-segment t-statistics for every m/z feature; these t-statistics measure the association between each ion's intensity and segment membership. Rank the m/z peaks within the target segment (liver) and comparison segment (heart) by their t-statistics, labeling peaks with positive t-statistics as enriched (higher abundance in that segment) and peaks with negative t-statistics as absent (lower abundance relative to the mean spectrum). Write the ranked enriched and absent peak tables per segment to CSV files. This step follows Cardinal's image segmentation framework, which partitions tissue into regions of homogeneous chemical composition and identifies the subset of informative ions, so the t-statistics are meaningful only after segmentation has been fit. Validate the output against the reported gold-standard counts: 92 enriched and 153 absent peaks for liver, and 23 enriched and 0 absent peaks for heart.

## Related tools

- **Cardinal** (R package providing the fitted segmentation and the topFeatures() t-statistic computation used to rank enriched and absent m/z peaks per segment) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment for loading the segmentation, running topFeatures(), ranking peaks, and writing CSV tables)

## Examples

```
BiocManager::install("Cardinal"); library(Cardinal)
```

## Evaluation signals

- Liver enriched-peak table contains exactly 92 peaks and liver absent-peak table contains exactly 153 peaks, matching the reported counts.
- Heart segment table contains exactly 23 enriched and 0 absent peaks.
- Every peak labeled 'enriched' has a positive t-statistic (higher abundance in the segment) and every 'absent' peak has a negative t-statistic (lower abundance vs. the mean spectrum).
- Both enriched and absent tables are ranked by t-statistic within each segment and exported as CSV files.
- The m/z features analyzed correspond to the peak-picked features produced by Cardinal's peakPick/peakAlign pre-processing.

## Limitations

- Counts are segment-specific and dataset-specific (92/153 for liver, 23/0 for heart in the pig fetus data); other tissues or segmentations will yield different enriched/absent tallies.
- Cardinal has dropped legacy support in Cardinal 3, so workflows written for older versions may break; the backend (matter) and pre-processing APIs were rewritten in recent versions.
- Interpretation depends on segmentation quality — uncertainty in the number of segments and selected informative ions propagates into the per-segment t-statistics.

## Evidence

- [abstract] Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues.: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [abstract] Image segmentation partitions a tissue into regions of homogeneous chemical composition: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition"
- [intro] Peak detection summarization is provided by peakPick for peak detection and summarization: "`peakPick()` for peak detection and summarization"
- [readme] The package includes classification and clustering methods based on nearest shrunken centroids: [evidence span withheld — anchor status: external_doc]
- [abstract] Image classification assigns locations on the tissue to pre-defined classes: "image classification, which assigns locations on the tissue to pre-defined classes"
