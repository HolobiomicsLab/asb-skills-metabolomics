---
name: longitudinal-intensity-trend-visualization
description: Use when a series of mzML files (e.g. pooled QC injections spanning weeks)
  needs per-compound intensity timelines to reveal drift, degradation, or sensitivity
  loss across batches.
when_to_use_negative:
- When the analysis goal is postacquisition drift correction of feature intensities
  rather than monitoring/QC visualisation - use a correction tool such as QC-MXP instead
- When files are not sequential injections of the same tracked compound list (e.g.
  one-off DDA acquisitions) - the time-line trend is meaningless without longitudinal
  QC samples
- When the input is already a processed feature/abundance table rather than mzML files
  needing peak detection by module_TrackCmp
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: Shiny web application and R package that registers new mzML files, runs module_TrackCmp
    peak detection for the user-defined m/z/RT pairs, and presents the intensity time
    line on a web dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: module_Files
  role: Looks for newly acquired files (mzML, mzXML, mzData etc.) by traversing a
    directory or reading from a text file containing file paths, then adds them to
    the database
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: module_TrackCmp
  role: Detects peaks based on a user-defined list of m/z and RT pairs, extracting
    observed m/z, RT, intensity and peak shape for each file
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Underlying language of the QC4Metabolomics R package; ggplot2/plotly used
    to draw the intensity time-line plot
- name: Shiny
  role: Web framework hosting the QC4Metabolomics dashboard where the longitudinal
    intensity trend is displayed
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:5d5a35322052a9e917d980cc83ccb345c8d5d3ecef96a34d1ab7be17e28d269f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  - Shiny
  - module_Files
  - module_TrackCmp
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Longitudinal intensity trend visualization

## Summary

Tracks the signal intensity of user-defined compounds (or contaminants) across sequentially acquired LC-MS files over time and renders the trend on a QC dashboard plot, so that intra- and interbatch drift, sensitivity loss, or degradation become visible for metabolomics QC.

## When to use

Use when a series of mzML files (e.g. pooled QC injections spanning weeks) needs per-compound intensity timelines to reveal drift, degradation, or sensitivity loss across batches.

## When NOT to use

- When the analysis goal is postacquisition drift correction of feature intensities rather than monitoring/QC visualisation - use a correction tool such as QC-MXP instead
- When files are not sequential injections of the same tracked compound list (e.g. one-off DDA acquisitions) - the time-line trend is meaningless without longitudinal QC samples
- When the input is already a processed feature/abundance table rather than mzML files needing peak detection by module_TrackCmp

## Inputs

- Vendor raw LC-MS metabolomics files converted to mzML (mzXML/mzData also accepted) covering sequential QC injections over the monitoring period
- User-defined list of m/z and retention-time (RT) pairs for the compound(s) to track
- Sample-name/keyword filter selecting the long-term QC sample injections

## Outputs

- Per-file tracked compound records: observed m/z, retention time, intensity, and peak shape diagnostics
- Time-line intensity plot (e.g. tryptophan_intensity_plot.png) showing intra- and interbatch intensity effects
- Dashboard view presented by the QC4Metabolomics Shiny web application

## How to apply

Feed the acquired mzML files into QC4Metabolomics so module_Files registers newly acquired files (by traversing a directory or reading a text file of paths) into the database. Define a list of m/z and retention-time (RT) pairs for the compound of interest, e.g. tryptophan, and let module_TrackCmp detect its peak in each file, extracting observed m/z, RT, intensity, and peak shape diagnostics. Apply sample-name filters to select only the long-term QC injections across all batches in the period. Plot the extracted intensities on a time line (ggplot2/plotly with viridis colour gradients), separating points by batch so that within-batch decrease and between-batch differences are both visible; the paper's example showed tryptophan intensity decreasing over a three-week period in long-term QC samples, clearly exposing both intra- and interbatch effects. Validate by checking that the plotted trend reproduces the reported intensity decrease and that the between-batch separation is consistent with the RT/m/z deviation figures (shorter-term RT deviations normally <0.05 min, batch differences at least twice as large).

## Related tools

- **QC4Metabolomics** (Shiny web application and R package that registers new mzML files, runs module_TrackCmp peak detection for the user-defined m/z/RT pairs, and presents the intensity time line on a web dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **module_Files** (Looks for newly acquired files (mzML, mzXML, mzData etc.) by traversing a directory or reading from a text file containing file paths, then adds them to the database) — https://github.com/stanstrup/QC4Metabolomics
- **module_TrackCmp** (Detects peaks based on a user-defined list of m/z and RT pairs, extracting observed m/z, RT, intensity and peak shape for each file) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Underlying language of the QC4Metabolomics R package; ggplot2/plotly used to draw the intensity time-line plot)
- **Shiny** (Web framework hosting the QC4Metabolomics dashboard where the longitudinal intensity trend is displayed) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
QC4Metabolomics::module_TrackCmp(mz_rt_list = c("Trp", 205.0972, 5.4)) # then plot intensity vs. injection time
```

## Evaluation signals

- The plotted time line must show the expected intensity trajectory, e.g. the tryptophan intensity decrease over the three-week period in long-term QC samples
- Batch separation must be visible: between-batch differences should exceed the within-batch variation, consistent with RT deviations <0.05 min short-term and at least twice as large between batches
- Every tracked file must yield a complete record (observed m/z, RT, intensity, peak shape) from module_TrackCmp; missing values flag failed peak detection
- Only files matching the QC sample filter (keyword/regex on sample name) should appear on the plot; biological sample injections must be excluded
- Output plot file (e.g. tryptophan_intensity_plot.png) must render and be reproducible from the same mzML inputs and tracked-compound list

## Limitations

- Requires vendor raw files to be converted to mzML (via ProteoWizard) and copied to a location QC4Metabolomics can access before tracking
- Detection depends on a user-supplied m/z/RT pair list; compounds not in the list are not tracked, and calibration drift can shift the observed m/z away from the target (e.g. the May/June 2022 calibration offset)
- Sensitivity changes (e.g. the June order-of-magnitude decrease) alter intensities and must be interpreted alongside precision, not as sample degradation alone
- The instruments will typically not alert the user if calibration is inaccurate, so visual inspection of the trend is required rather than relying on instrument warnings

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples: "the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples"
- [abstract] monitors files as they are acquired or retrospectively by tracking any user-defined compound(s): "QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s)"
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.): "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.)"
- [results] the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [abstract] presents the results on a web dashboard: "and presents the results on a web dashboard"
