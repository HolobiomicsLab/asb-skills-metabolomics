---
name: spatial-segment-tstatistic-extraction
description: Use when you have a fitted Cardinal segmentation of an MSI tissue section
  (e.g. imzML-derived peak data from pig fetus liver/heart sections) and need ranked
  lists of peaks enriched or absent in each spatial segment.
when_to_use_negative:
- The goal is supervised image classification into pre-defined classes with cross-validated
  error estimation rather than characterizing segments of an unsupervised segmentation.
- Input is raw or unprocessed spectra — Cardinal's pre-processing (normalize, smooth,
  reduceBaseline, peakPick/peakAlign) must be run first so topFeatures() operates
  on aligned peak features.
- No fitted segmentation exists yet; topFeatures() requires the fitted segmentation
  object as input.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Cardinal
  role: R package whose fitted segmentation objects and topFeatures() function provide
    the per-segment t-statistics used to rank and label peaks as enriched or absent
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment for loading the fitted segmentation, running topFeatures(),
    and writing ranked enriched/absent peak tables to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:8f092328ee4a0d9749289d32ec31fb7ac07d0bca5547def1ec3ac1b429af8d52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spatial-segment-tstatistic-extraction

## Summary

Extracts per-segment differential ion statistics from a fitted Cardinal spatial segmentation by applying topFeatures() to compute t-statistics for every m/z feature, then labeling each peak per segment as enriched (positive t) or absent (negative t) relative to the mean spectrum.

## When to use

Use when you have a fitted Cardinal segmentation of an MSI tissue section (e.g. imzML-derived peak data from pig fetus liver/heart sections) and need ranked lists of peaks enriched or absent in each spatial segment.

## When NOT to use

- The goal is supervised image classification into pre-defined classes with cross-validated error estimation rather than characterizing segments of an unsupervised segmentation.
- Input is raw or unprocessed spectra — Cardinal's pre-processing (normalize, smooth, reduceBaseline, peakPick/peakAlign) must be run first so topFeatures() operates on aligned peak features.
- No fitted segmentation exists yet; topFeatures() requires the fitted segmentation object as input.

## Inputs

- Fitted Cardinal segmentation result object for an MSI tissue dataset (e.g. liver/heart pig fetus sections)
- Peak-picked and aligned m/z feature intensities per pixel (Cardinal peakPick/peakAlign/peakProcess output)
- Segment labels for which differential statistics are needed (e.g. liver, heart)

## Outputs

- Ranked table of enriched peaks for the liver segment (positive t-statistics), exported as CSV
- Ranked table of absent peaks for the liver segment (negative t-statistics), exported as CSV
- Ranked enriched/absent peak table for the heart segment, exported as CSV
- Per-segment t-statistics for every m/z feature from topFeatures()

## How to apply

Load the fitted segmentation result in R with the Cardinal package, then apply topFeatures() to the fitted segmentation so that per-segment t-statistics are computed for every m/z feature; these statistics measure the association between each ion's intensity and segment membership. Rank the m/z peaks within each segment of interest (e.g. liver and heart) by t-statistic, and label each peak enriched when its t-statistic is positive (higher abundance in that segment) or absent when negative (lower abundance in that segment). Write the ranked enriched and absent peak tables per segment to CSV. The labeling direction (sign of the t-statistic relative to the segment versus the mean spectrum) is the key decision point, and the segmentation itself should already have come from Cardinal's mixture-model/regularization-based image segmentation on preprocessed peak-picked spectra.

## Related tools

- **Cardinal** (R package whose fitted segmentation objects and topFeatures() function provide the per-segment t-statistics used to rank and label peaks as enriched or absent) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment for loading the fitted segmentation, running topFeatures(), and writing ranked enriched/absent peak tables to CSV)

## Examples

```
library(Cardinal); tf <- topFeatures(fitted_segmentation, n=Inf); liver_enriched <- subset(tf, class=='liver' & statistic > 0)
```

## Evaluation signals

- Liver enriched table contains exactly 92 peaks and liver absent table contains exactly 153 peaks, matching the reported counts for the pig fetus segmentation.
- Heart segment table contains exactly 23 enriched peaks and 0 absent peaks.
- Sign convention holds: every peak in an enriched table has a positive t-statistic and every peak in an absent table has a negative t-statistic for that segment.
- Output CSVs exist for both liver and heart segments and contain ranked m/z features with their t-statistics.
- t-statistics were computed over the full m/z feature set (topFeatures() applied to the fitted segmentation), not a pre-filtered subset.

## Limitations

- Counts are dataset-specific (92 enriched / 153 absent liver, 23 enriched / 0 absent heart); other datasets will yield different numbers, so exact-count validation only transfers to this pig fetus liver/heart MSI dataset.
- Interpretation of enriched vs. absent is relative to the mean spectrum of the fitted segmentation, not an external control.
- Cardinal's statistical methods are based on mixture modeling and regularization, so results depend on the number of segments chosen and the quality of the upstream segmentation.

## Evidence

- [abstract] topFeatures-based t-statistic extraction workflow: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [abstract] segmentation partitions tissue into homogeneous chemical regions: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions"
- [readme] nearest shrunken centroids methods underpin segmentation/classification: [evidence span withheld — anchor status: external_doc]
- [intro] peak pre-processing requirement for feature statistics: "`peakPick()` for peak detection and summarization"
