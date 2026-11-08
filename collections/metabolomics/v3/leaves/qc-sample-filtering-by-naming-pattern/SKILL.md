---
name: qc-sample-filtering-by-naming-pattern
description: Use when a QC4Metabolomics-style dashboard must distinguish long-term
  QC samples from biological or calibrant injections across multiple batches before
  extracting intensity, RT or m/z trends.
when_to_use_negative:
- When the goal is to monitor a user-defined compound in a single targeted sample
  rather than selecting QC samples from a multi-batch file table — use module_TrackCmp
  directly.
- When naming conventions are absent or inconsistent (e.g. samples acquired under
  ad-hoc naming) so that a keyword/regex filter cannot reliably separate QC from biological
  injections.
- When correcting intensity drifts postanalysis rather than filtering which samples
  to monitor — QC-MXP-style correction tools are the appropriate skill.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: 'Hosts the filtering logic: module_Files, module_File_info and module_TrackCmp
    operate on the filtered QC sample set to produce QC-parameter trends.'
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web front-end where the filtered QC records are displayed as dashboards with
    viridis colour gradients and batch-separated timelines.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the QC4Metabolomics package that performs the keyword/regex
    filtering on sample names.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment vehicle for the QC4Metabolomics pipeline (including the ProteoWizard
    converter feeding the file table) that the filtering step consumes.
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:c06eb2bcaf0ae366f77a586f734e142d7de06fc008b9e4794922b08cb5818a55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# QC sample filtering by naming pattern

## Summary

Selects the correct QC/control injections from a metabolomics file table by applying keyword or regular-expression filters to sample names, so that QC-parameter tracking (e.g. compound-intensity drifts over weeks) is computed on the right subset of long-term QC samples.

## When to use

Use when a QC4Metabolomics-style dashboard must distinguish long-term QC samples from biological or calibrant injections across multiple batches before extracting intensity, RT or m/z trends.

## When NOT to use

- When the goal is to monitor a user-defined compound in a single targeted sample rather than selecting QC samples from a multi-batch file table — use module_TrackCmp directly.
- When naming conventions are absent or inconsistent (e.g. samples acquired under ad-hoc naming) so that a keyword/regex filter cannot reliably separate QC from biological injections.
- When correcting intensity drifts postanalysis rather than filtering which samples to monitor — QC-MXP-style correction tools are the appropriate skill.

## Inputs

- File table produced by module_Files (mzML / mzXML / mzData paths plus metadata from module_File_info)
- Sample-name column from module_File_info (instrument name, project name, ionization mode, sample name)
- Keyword or regular-expression pattern defining the long-term QC sample naming convention

## Outputs

- Filtered subset of QC sample records (per-file rows restricted to long-term QC injections across all batches)
- Per-file QC-parameter records suitable for module_TrackCmp peak tracking (observed m/z, RT, intensity, peak shape)

## How to apply

After the file table is populated (module_Files ingests mzML/mzXML/mzData paths and module_File_info extracts instrument name, project name, ionization mode and sample name), apply keyword or regular-expression filters on the sample-name column to keep only long-term QC injections spanning all batches in the monitoring period. The filtered subset is then handed to module_TrackCmp, which detects peaks from a user-defined list of m/z and retention time (RT) pairs and extracts observed m/z, retention time, intensity and peak shape; in the paper this pattern was used to isolate three weeks of long-term QC samples for tryptophan monitoring. Filter specificity matters: if the pattern also matches calibrant or pooled-QC-short runs, intensity trends become confounded, so test the filter against the file-info table first. The result is a per-file QC record set whose trends (e.g. the reported decrease in tryptophan intensity over three weeks with intra- and interbatch effects) can be visualised on the Shiny dashboard with viridis-coloured, batch-separated plots.

## Related tools

- **QC4Metabolomics** (Hosts the filtering logic: module_Files, module_File_info and module_TrackCmp operate on the filtered QC sample set to produce QC-parameter trends.) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web front-end where the filtered QC records are displayed as dashboards with viridis colour gradients and batch-separated timelines.) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the QC4Metabolomics package that performs the keyword/regex filtering on sample names.) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment vehicle for the QC4Metabolomics pipeline (including the ProteoWizard converter feeding the file table) that the filtering step consumes.) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
Rscript -e "library(QC4Metabolomics); qc <- filter(module_File_info()$db, grepl('long.term.QC', sample_name)); module_TrackCmp(qc, cmp_list)"
```

## Evaluation signals

- After filtering, every retained record should correspond to a long-term QC injection across all batches in the monitoring period; verify no biological or calibrant sample names remain.
- Plotting tryptophan intensity from the filtered set should reproduce the reported three-week decrease with clearly visible intra- and interbatch effects.
- Check that module_TrackCmp peak fields (observed m/z, RT, intensity, peak shape) are non-empty for each filtered file, indicating the correct QC subset was passed downstream.
- Re-run the filter with a slightly relaxed/strictened regex and confirm the number of retained rows changes only at true QC sample boundaries, not at batch edges.
- RT deviations in the filtered QC set should fall within the reported norms (short-term < 0.05 min; between-batch differences at least twice as large), which would be violated if the wrong sample class were retained.

## Limitations

- Filtering depends entirely on the sample-name convention; if a new batch uses different naming, previously retained QC samples may be missed or non-QC samples may be included.
- The skill does not itself correct calibration or intensity drift — it only selects which samples to monitor; the article warns that instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate.
- Contaminant-level monitoring (module_Contaminants, ~800 known contaminant ions) is a separate module and is not covered by naming-pattern QC filtering.

## Evidence

- [methods] module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name): "module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name)"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples: "the intensity has decreased during a three-week period for tryptophan measured in our long-term QC samples"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [abstract] QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics: "QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics"
