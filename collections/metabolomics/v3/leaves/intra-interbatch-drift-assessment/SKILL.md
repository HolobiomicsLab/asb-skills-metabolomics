---
name: intra-interbatch-drift-assessment
description: Use when monitoring a specific compound's intensity across long-term
  QC injections spanning multiple batches to quantify drift that occurs within a batch
  versus between batches.
when_to_use_negative:
- The data set is already a processed feature table with no raw mzML available for
  module_TrackCmp peak detection.
- Only a single batch is present, so no between-batch comparison can be made.
- The target compound is not on the user-defined m/z–RT list and no reference values
  are available.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Shiny web application and R package that registers files, runs module_TrackCmp
    for peak detection against user-defined m/z–RT pairs, and presents results on
    a web dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web dashboard framework that displays the tracking results and the compound-intensity
    time-line plot
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Underlying R package ecosystem used by QC4Metabolomics for file handling,
    peak extraction, and plotting (ggplot2/plotly)
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:691d360498708b3fccdfbd9b2da106dfc2d8dcb1bc0f6d603e65bc3d9ada3ae1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# intra-interbatch-drift-assessment

## Summary

A QC4Metabolomics-based skill for assessing intensity drift within and between batches by tracking user-defined compounds (e.g., tryptophan) across long-term pooled QC samples, using module_TrackCmp to extract m/z, retention time, intensity and peak-shape diagnostics from mzML files and plotting them on a time-line to reveal intra- and interbatch effects.

## When to use

Use when monitoring a specific compound's intensity across long-term QC injections spanning multiple batches to quantify drift that occurs within a batch versus between batches.

## When NOT to use

- The data set is already a processed feature table with no raw mzML available for module_TrackCmp peak detection.
- Only a single batch is present, so no between-batch comparison can be made.
- The target compound is not on the user-defined m/z–RT list and no reference values are available.

## Inputs

- Long-term pooled QC sample mzML files spanning the acquisition period (e.g., three weeks)
- User-defined list of m/z and retention time pairs for the target compound(s) (e.g., tryptophan)
- Database table of registered files from module_Files

## Outputs

- Compound tracking plot (e.g., tryptophan_intensity_plot.png) showing intensity vs. time with points separated by batch
- Extracted per-injection diagnostics: observed m/z, retention time, intensity, and peak shape for the tracked compound

## How to apply

1. Load the long-term QC sample mzML files covering the target acquisition period (e.g., a three-week run) into QC4Metabolomics; the module_Files step registers newly acquired files in the database. 2. Use module_TrackCmp to detect the target peak (e.g., tryptophan) against a user-defined list of m/z and retention time pairs, extracting observed m/z, retention time, intensity, and peak shape. 3. Filter injections to long-term QC samples using keyword/regular-expression filters across all batches in the period. 4. Plot the extracted intensities on a time-line (ggplot2/plotly with viridis colour gradients), separating points by batch so both intra- and interbatch effects are visible; the article shows the three-week tryptophan decrease with between-batch differences. 5. Validate that the plot shows the reported intensity decrease over the period and that between-batch offsets are consistent with the article's finding that batch differences can be at least twice the shorter-term (<0.05 min) RT deviation magnitude.

## Related tools

- **QC4Metabolomics** (Shiny web application and R package that registers files, runs module_TrackCmp for peak detection against user-defined m/z–RT pairs, and presents results on a web dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web dashboard framework that displays the tracking results and the compound-intensity time-line plot) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Underlying R package ecosystem used by QC4Metabolomics for file handling, peak extraction, and plotting (ggplot2/plotly)) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
Rscript -e "QC4Metabolomics::module_TrackCmp(mzml_dir='longterm_QC_mzML/', mz_rt_list=trp_list)"; then plot intensities coloured by batch (ggplot2/plotly, viridis).
```

## Evaluation signals

- The generated plot must show a monotonic decrease in tryptophan intensity across the three-week period for the long-term QC samples.
- Between-batch intensity offsets must be visible and larger than the within-batch variation, consistent with the article's finding that batch differences can be at least twice the shorter-term (<0.05 min) RT deviation magnitude.
- Every QC injection in the period must appear as a point in the plot; missing points indicate incomplete file registration or filter failures.
- Observed m/z and retention time for each extracted peak must fall within the tolerances defined in the user-supplied m/z–RT pair list.
- Peak shape diagnostics must be present for each detected peak to confirm that the extracted intensity corresponds to a real chromatographic peak rather than noise.

## Limitations

- The method relies on the target compound being present in the user-defined m/z–RT list; compounds outside this list are not tracked.
- Peak detection depends on accurate retention-time alignment; large RT shifts (e.g., >0.05 min) between batches may reduce matching reliability.
- The approach assumes that long-term pooled QC samples are injected at regular intervals; irregular QC spacing may confound the drift assessment.
- Only mzML/mzXML/mzData file formats are supported by the file-registration module.

## Evidence

- [results] The plot clearly showed both intra- and interbatch effects.: "the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples"
- [results] Batch-level differences are larger than within-batch variation.: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] module_TrackCmp detects peaks based on user-defined m/z–RT pairs.: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [abstract] The tool presents the tracking results on a web dashboard.: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [results] Long-term QC pool data used for drift monitoring.: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
