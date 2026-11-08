---
name: chromatographic-peak-tracking-for-user-defined-compounds
description: Use when you have raw untargeted metabolomics files (mzML/mzXML/mzData)
  plus a user-defined list of m/z and RT pairs for QC or contaminant compounds, and
  need to track observed m/z, retention time, intensity, and peak shape over time
  or batches.
when_to_use_negative:
- When the compound list is a set of contaminant ions to monitor generically — module_Contaminants
  (nearly 800 known contaminant ions) is the dedicated path, not a bespoke m/z-RT
  tracking list
- When the goal is intensity-drift correction or normalization of the full feature
  table (e.g. QC-MXP-style correction) rather than targeted tracking of a few compounds
- When the input is already a processed peak/feature table with no raw mzML files
  to extract peaks from
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: Hosts module_TrackCmp, which detects peaks from the user-defined m/z and RT
    list and supplies the monitoring database and dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web application layer that presents the tracked-compound results on a dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Package language in which the tracking module and plotting (ggplot2/plotly)
    are implemented
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Runs the ProteoWizard converter that produces the mzML inputs consumed by
    tracking
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:d0d597c6a41b1a864c1dae02c30df7d439cfc55d3194823f5095560ecea91b14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chromatographic-peak-tracking-for-user-defined-compounds

## Summary

Detects and time-lines the intensity of user-specified compounds (m/z + RT pairs) across mzML injections using the QC4Metabolomics module_TrackCmp workflow, revealing drifts such as the three-week tryptophan intensity decrease in long-term QC samples.

## When to use

Use when you have raw untargeted metabolomics files (mzML/mzXML/mzData) plus a user-defined list of m/z and RT pairs for QC or contaminant compounds, and need to track observed m/z, retention time, intensity, and peak shape over time or batches.

## When NOT to use

- When the compound list is a set of contaminant ions to monitor generically — module_Contaminants (nearly 800 known contaminant ions) is the dedicated path, not a bespoke m/z-RT tracking list
- When the goal is intensity-drift correction or normalization of the full feature table (e.g. QC-MXP-style correction) rather than targeted tracking of a few compounds
- When the input is already a processed peak/feature table with no raw mzML files to extract peaks from

## Inputs

- Vendor raw MS metabolomics data converted to mzML (or direct mzML/mzXML/mzData acquisitions)
- User-defined list of m/z and retention time (RT) pairs for compounds to track
- Sample-name keyword/regular-expression filters selecting the injections to monitor (e.g. long-term QC pool)

## Outputs

- Per-injection table of observed m/z, retention time, intensity, and peak-shape diagnostics for each tracked compound
- Time-line plot of tracked compound intensity across injections/batches (e.g. tryptophan_intensity_plot.png)
- Retrospective view of intra- and interbatch intensity effects

## How to apply

Register each acquired file (vendor raw converted to mzML via ProteoWizard in Docker, or mzML/mzXML/mzData found by module_Files) in the QC4Metabolomics database, then filter injections with keyword/regular-expression sample-name matching (e.g. to select the long-term QC pool such as MetNexs). Run module_TrackCmp with the user-defined m/z and RT list to extract observed m/z, retention time, intensity, and peak-shape diagnostics per injection. Plot the resulting intensity series on a time-line (ggplot2/plotly with viridis gradients), coloring or faceting by batch so that both intra- and interbatch effects are visible — e.g. the reported case where tryptophan intensity in long-term QC samples decreased over a three-week period. Judge success by the plot reproducing the monotone decrease and between-batch offsets for the tracked compound.

## Related tools

- **QC4Metabolomics** (Hosts module_TrackCmp, which detects peaks from the user-defined m/z and RT list and supplies the monitoring database and dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web application layer that presents the tracked-compound results on a dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Package language in which the tracking module and plotting (ggplot2/plotly) are implemented) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Runs the ProteoWizard converter that produces the mzML inputs consumed by tracking) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
Rscript module_TrackCmp.R --mzml longterm_QC/*.mzML --targets m-z_rt_list.csv --filter-regex "MetNexs"
```

## Evaluation signals

- The extracted peak table contains observed m/z and RT per injection that deviate from the user-defined targets by amounts consistent with article-reported tolerances (short-term RT deviation < 0.05 min; between-batch deviations at least twice that)
- For the reference demo, the time-line plot of tryptophan in long-term QC samples shows a monotone decrease over the three-week period
- The plot separates points by batch, exposing interbatch offsets in addition to the within-batch decline
- Tracked file set matches the keyword/regular-expression sample-name filter applied to the QC pool (e.g. MetNexs) and excludes other injections

## Limitations

- The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate — tracked m/z can drift silently; calibration offsets must be diagnosed from the tracked data itself (e.g. the May/June 2022 offset)
- Ion suppression or enhancement affecting only certain peaks, nonobvious contaminants, and loss of calibration in a limited mass region are signals untargeted QC alone can miss; targeted tracking must be paired with contaminant and warner modules to catch them
- Tracking relies on user-supplied m/z and RT pairs; if a compound is absent from the list it is not monitored, and RT or m/z drifts beyond the user-defined window may cause missed peak detection

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [results] The intensity has decreased during a three-week period for tryptophan measured in long-term QC samples: "the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples"
- [results] Short-term RT deviations normally < 0.05 min while between-batch differences can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [abstract] QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics: "QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics"
- [abstract] The Shiny app is available as open-source code: "The Shiny app is available as open-source code"
