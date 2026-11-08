---
name: mz-rt-pair-matching-in-qc-samples
description: Use when monitoring specific compounds (e.g. tryptophan, calibrant, contaminants)
  in pooled/long-term QC samples across batches, or when you need per-file intensity/RT/peak-shape
  diagnostics from untargeted metabolomics acquisitions.
when_to_use_negative:
- The input is an already extracted feature table (e.g. from XCMS/CAMERA peak picking)
  rather than vendor raw or mzML spectra — use post-analysis drift correction instead.
- The goal is global untargeted feature detection rather than monitoring a pre-defined
  list of m/z–RT pairs.
- Spectra are not in a ProteoWizard-compatible vendor format or mzML/mzXML/mzData
  container that module_Files can ingest.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QC4Metabolomics
  role: R package / Shiny app providing module_TrackCmp, the m/z–RT pair matching
    engine and downstream database and dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the matching modules
- name: Shiny
  role: Web dashboard that plots matched compound intensities and RT deviations per
    batch
- name: ProteoWizard
  role: Converts vendor raw data to mzML before matching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:5d66a5c890960b29b237095096b25eedab8c05fbaf83ddc435f3b87d3ab8b204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - ProteoWizard
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# m/z–RT pair matching in QC samples

## Summary

Detects user-defined compounds in QC sample mzML files by matching observed peaks against a supplied list of m/z and retention-time pairs, extracting m/z, RT, intensity, and peak-shape diagnostics for tracking over time. It is the core peak-detection routine (module_TrackCmp) that feeds QC4Metabolomics compound-tracking plots.

## When to use

Use when monitoring specific compounds (e.g. tryptophan, calibrant, contaminants) in pooled/long-term QC samples across batches, or when you need per-file intensity/RT/peak-shape diagnostics from untargeted metabolomics acquisitions.

## When NOT to use

- The input is an already extracted feature table (e.g. from XCMS/CAMERA peak picking) rather than vendor raw or mzML spectra — use post-analysis drift correction instead.
- The goal is global untargeted feature detection rather than monitoring a pre-defined list of m/z–RT pairs.
- Spectra are not in a ProteoWizard-compatible vendor format or mzML/mzXML/mzData container that module_Files can ingest.

## Inputs

- mzML (or mzXML, mzData) files of QC / pooled QC samples, including long-term QC injections across multiple batches
- user-defined list of compounds with expected m/z and retention time (RT) pairs and matching tolerances
- database or text file of newly acquired file paths registered by module_Files

## Outputs

- per-file peak detections: observed m/z, retention time, intensity and peak shape for each matched m/z–RT pair
- database rows keyed by file/compound used for time-series plotting (e.g. tryptophan intensity over three weeks of QC samples)
- input table for dashboard plots showing intra- and interbatch intensity/RT effects

## How to apply

Supply a table of known compounds with expected m/z and retention time (RT) pairs plus mass and RT tolerance windows. module_TrackCmp scans each mzML file (converted from vendor raw data with ProteoWizard) and detects peaks whose measured m/z and RT fall within those windows, extracting the observed m/z, retention time, intensity, and peak-shape information. It runs as part of the QC4Metabolomics R package pipeline after module_Files has registered newly acquired files in the database, and results are written to a database table for plotting on the Shiny dashboard. The same matching underlies module_Contaminants, which uses a pre-built list of nearly 800 known contaminant ions. Tolerances should be set per compound; RT deviations of <0.05 min are normal short-term while between-batch differences can be at least twice as large, so batch-aware RT windows are recommended for multi-week acquisitions.

## Related tools

- **QC4Metabolomics** (R package / Shiny app providing module_TrackCmp, the m/z–RT pair matching engine and downstream database and dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the matching modules)
- **Shiny** (Web dashboard that plots matched compound intensities and RT deviations per batch)
- **ProteoWizard** (Converts vendor raw data to mzML before matching)

## Examples

```
Rscript -e "QC4Metabolomics::module_TrackCmp(mz_rt_pairs = cmp_list, files = mzml_files)"
```

## Evaluation signals

- Each user-defined m/z–RT pair yields a detection in files where the compound is present, with non-null observed m/z, RT, intensity and peak-shape fields.
- Observed m/z and RT values fall inside the specified matching windows; RT deviations for short-term within-batch QC runs should be <0.05 min.
- Reproduced time-series (e.g. tryptophan in long-term QC over three weeks) must show the expected trends, such as intensity decrease and clear intra-/interbatch effects.
- Matched rows are persisted in the QC4Metabolomics database and resurface on the Shiny dashboard without manual re-entry.
- Contaminant matching (module_Contaminants) flags ions from the ~800-entry contaminant list when they exceed user-defined thresholds.

## Limitations

- Matching depends on files having been registered by module_Files and converted to mzML; files that skip these steps are not matched.
- RT windows must be widened or batch-aware when comparing across batches, since between-batch RT differences can be at least twice the within-batch (<0.05 min) deviations.
- Sensitivity losses (e.g. the order-of-magnitude drop observed in June 2022) reduce peak detection and downstream precision.
- The instruments themselves do not alert or stop acquisition when calibration is inaccurate, so unmatched or shifted peaks may go unnoticed without real-time monitoring.

## Evidence

- [methods] Peak detection matched against user-defined m/z and RT pairs: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [abstract] Diagnostic information extracted per matched compound: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [methods] Files must be converted to mzML and registered by module_Files before matching: "module_Files looks for newly acquired
files (mzML, mzXML, mzData etc.)"
- [results] Within-batch RT deviation threshold that informs tolerance settings: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] Contaminant matching extends the same pair-matching concept: "module_Contaminants uses
a list of
nearly 800 known contaminant ions"
- [readme] Repo URL grounding related_tools: "QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics"
