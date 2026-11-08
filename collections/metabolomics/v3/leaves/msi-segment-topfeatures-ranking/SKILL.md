---
name: msi-segment-topfeatures-ranking
description: Use when an imzML/Analyze 7.5 MSI dataset has already been segmented
  (e.g. pig fetus liver/heart sections) and you need the per-segment enriched/absent
  peak counts and ranked peak tables.
when_to_use_negative:
- Input is raw spectra or a pre-processed imzML/Analyze 7.5 file that has not yet
  been segmented — run Cardinal image segmentation first, since topFeatures() operates
  on a fitted segmentation
- The goal is assigning pixels to pre-defined classes (image classification) rather
  than characterizing which ions distinguish segments of an unsupervised/supervised
  segmentation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Cardinal
  role: R package providing the fitted segmentation object and the topFeatures() function
    used to compute per-segment t-statistics for every m/z feature
  repo: https://github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment in which the Cardinal segmentation is loaded, topFeatures()
    is applied, and ranked peak tables are written to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:9ac8cc11806b3371f2aec0df96d5d9cab28f9e1dcfaa764e10e2ee445754e817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuwisdelu/Cardinal
  tools:
  - Cardinal
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msi-segment-topfeatures-ranking

## Summary

Rank m/z peaks within each tissue segment of a fitted Cardinal spatial-shrunken-centroids segmentation by per-segment t-statistics from topFeatures(), labeling peaks as enriched (positive t) or absent (negative t) relative to the mean spectrum.

## When to use

Use when an imzML/Analyze 7.5 MSI dataset has already been segmented (e.g. pig fetus liver/heart sections) and you need the per-segment enriched/absent peak counts and ranked peak tables.

## When NOT to use

- Input is raw spectra or a pre-processed imzML/Analyze 7.5 file that has not yet been segmented — run Cardinal image segmentation first, since topFeatures() operates on a fitted segmentation
- The goal is assigning pixels to pre-defined classes (image classification) rather than characterizing which ions distinguish segments of an unsupervised/supervised segmentation

## Inputs

- Fitted Cardinal image segmentation result (e.g. from spatial shrunken centroids on an imzML or Analyze 7.5 mass spectrometry imaging dataset)
- Segment identities of interest (e.g. liver and heart segments of the pig fetus tissue sections)

## Outputs

- Ranked enriched-peak table per segment (m/z features with positive t-statistics, exported to CSV)
- Ranked absent-peak table per segment (m/z features with negative t-statistics, exported to CSV)
- Per-segment counts of systematically enriched and systematically absent peaks (e.g. liver: 92 enriched / 153 absent; heart: 23 enriched / 0 absent)

## How to apply

Load the fitted segmentation result for the MSI dataset in R using the Cardinal package, then call topFeatures() on the fitted segmentation to compute t-statistics for every m/z feature; these t-statistics measure the association between each ion's intensity and segment membership. Rank the m/z peaks within each segment of interest (e.g. liver, heart) by their t-statistics, and label each peak as enriched when its t-statistic is positive (higher abundance in that segment) or absent when negative (lower abundance in that segment). Write the ranked enriched and absent peak tables per segment to CSV. Validate against known counts where available: for the pig fetus liver segment expect exactly 92 enriched and 153 absent peaks, and for the heart segment expect 23 enriched and 0 absent peaks, matching the reported counts.

## Related tools

- **Cardinal** (R package providing the fitted segmentation object and the topFeatures() function used to compute per-segment t-statistics for every m/z feature) — https://github.com/kuwisdelu/Cardinal
- **R** (Runtime environment in which the Cardinal segmentation is loaded, topFeatures() is applied, and ranked peak tables are written to CSV)

## Examples

```
library(Cardinal); tf <- topFeatures(fitted_segmentation); liver <- tf[tf$segment == "liver",]; enriched <- liver[liver$t > 0,]; absent <- liver[liver$t < 0,]; write.csv(enriched, "liver_enriched.csv"); write.csv(absent, "liver_absent.csv")
```

## Evaluation signals

- topFeatures() returns a finite t-statistic for every m/z feature of the fitted segmentation
- Sign convention holds: peaks labeled enriched have positive t-statistics (higher abundance in that segment) and peaks labeled absent have negative t-statistics (lower abundance)
- For the pig fetus dataset, the liver tables contain exactly 92 enriched and 153 absent peaks
- For the pig fetus dataset, the heart table contains exactly 23 enriched and 0 absent peaks
- Ranked peak tables per segment are written to CSV and each row corresponds to one m/z feature with its t-statistic

## Limitations

- Cardinal 3 dropped legacy support and merged/deprecated redundant functions, so topFeatures() usage must match the installed Cardinal version (e.g. Cardinal 3.6 with the rewritten matter backend)
- Counts are dataset-specific: the 92/153 (liver) and 23/0 (heart) enriched/absent totals validate only the pig fetus segmentation, not other tissues or segment numbers
- Interpretation of t-statistics depends on the segmentation model being based on mixture modeling and regularization, so results change if the number of segments or the informative ion subset changes

## Evidence

- [other] t-statistics quantified the relative importance of peaks in the liver: 92 peaks were systematically enriched and 153 were systematically absent compared with the mean spectrum.: [evidence span withheld — anchor status: not_found]
- [other] Apply topFeatures() to the fitted segmentation to compute per-segment t-statistics for every m/z feature: [evidence span withheld — anchor status: not_found]
- [abstract] image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions"
- [intro] *Cardinal* natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats via the `readMSIData()`: "*Cardinal* natively supports reading and writing imzML (both "continuous" and "processed" types) and Analyze 7.5 formats"
