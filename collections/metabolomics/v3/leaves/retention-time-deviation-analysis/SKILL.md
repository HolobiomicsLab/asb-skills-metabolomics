---
name: retention-time-deviation-analysis
description: Use when you need to quantify retention time stability of monitored compounds
  in a long-term QC pool (e.g. MetNexs) across LC-MS batches, comparing short-term
  within-batch deviations against interbatch offsets using QC4Metabolomics.
when_to_use_negative:
- When you only need intensity or peak-shape drift monitoring rather than RT stability
  (module_TrackCmp extracts multiple diagnostics; this skill targets RT deviations
  specifically).
- When the analysis goal is postanalysis intensity-drift correction rather than RT
  deviation quantification (QC-MXP-style correction is a different workflow).
- When you have no batch-level grouping or reference RTs available, since the two-regime
  comparison cannot be computed.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Open-source Shiny web application and R package whose module_TrackCmp detects
    peaks from the user-defined m/z/RT pair list and produces the RT deviation data
    used for within-batch and interbatch comparison.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Underlying language of the QC4Metabolomics package; used to compute batch-grouped
    RT deviation statistics.
- name: Shiny
  role: Web framework through which QC4Metabolomics presents the RT deviation results
    on a dashboard.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:84f4a11cbe97dc80f5ae2f7e5d860e1c337ad1054022be53986053f075a39f2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# retention-time-deviation-analysis

## Summary

Tracks user-defined m/z and retention time (RT) pairs with QC4Metabolomics module_TrackCmp to quantify within-batch versus interbatch RT deviations for a long-term QC pool, distinguishing the short-term (< 0.05 min) regime from batch-to-batch offsets at least twice as large.

## When to use

Use when you need to quantify retention time stability of monitored compounds in a long-term QC pool (e.g. MetNexs) across LC-MS batches, comparing short-term within-batch deviations against interbatch offsets using QC4Metabolomics.

## When NOT to use

- When you only need intensity or peak-shape drift monitoring rather than RT stability (module_TrackCmp extracts multiple diagnostics; this skill targets RT deviations specifically).
- When the analysis goal is postanalysis intensity-drift correction rather than RT deviation quantification (QC-MXP-style correction is a different workflow).
- When you have no batch-level grouping or reference RTs available, since the two-regime comparison cannot be computed.

## Inputs

- User-defined list of m/z and retention time (RT) pairs for compounds to track (e.g. MetNexs QC pool compounds)
- Long-term QC pool LC-MS data across all acquisition batches, with batch identifiers per injection
- Reference retention times for the monitored compounds

## Outputs

- Per-compound RT deviation values with batch labels for each injection
- Within-batch (short-term) RT deviation statistics per monitored compound
- Between-batch RT deviation statistics relative to reference RTs
- Two-regime summary comparison table (within-batch vs interbatch)

## How to apply

Load the RT deviation data for the long-term QC pool (e.g. MetNexs) analyzed across all batches, as tracked by module_TrackCmp from a user-defined list of m/z and retention time (RT) pairs. Group the RT deviations by batch identifier so each injection's deviation is attributed to its acquisition batch, enabling within-batch and between-batch comparisons for the same compounds. Compute within-batch (short-term) RT deviation statistics per monitored compound, then compute between-batch RT deviation statistics by comparing batch-level RT offsets against the reference RTs. Verify the two-regime result: within-batch deviations should normally be less than 0.05 min, while differences between different batches can be at least twice as large. Save the per-compound RT deviations with batch labels and the two-regime summary as a structured table for downstream QC reporting.

## Related tools

- **QC4Metabolomics** (Open-source Shiny web application and R package whose module_TrackCmp detects peaks from the user-defined m/z/RT pair list and produces the RT deviation data used for within-batch and interbatch comparison.) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Underlying language of the QC4Metabolomics package; used to compute batch-grouped RT deviation statistics.)
- **Shiny** (Web framework through which QC4Metabolomics presents the RT deviation results on a dashboard.)

## Examples

```
Rscript -e 'library(QC4Metabolomics); dev <- read_csv("metnexs_rt_deviation.csv"); dev %>% group_by(batch) %>% summarise(mean_rt_dev = mean(rt_dev))'
```

## Evaluation signals

- Reproduced summary shows within-batch RT deviations normally below 0.05 min for the QC pool.
- Interbatch RT differences are at least twice the within-batch magnitude (two-regime result).
- Every deviation value is labeled with its acquisition batch so within-batch and between-batch groupings are unambiguous.
- Per-compound RT deviations are traceable to module_TrackCmp output from the user-defined m/z/RT pair list.
- Output is a structured table carrying batch labels and the two-regime comparison.

## Limitations

- The 0.05 min threshold and 'at least twice as large' interbatch scaling are empirical observations from the MetNexs long-term QC pool; they are not guaranteed generalizable to other instruments or QC pools.
- Calibration or instrument drift (e.g. the May/June 2022 offset, calibrant signal loss) can confound RT deviations if not accounted for.
- Instrument typically does not alert the user when calibration is inaccurate, so RT deviations may reflect calibration problems rather than chromatographic instability.

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] QC4Metabolomics leverages the rich ecosystem of R packages: "QC4Metabolomics leverages the rich ecosystem of R packages"
- [abstract] extracting diagnostic information such as observed m/z, retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z, retention time, intensity and peak shape"
