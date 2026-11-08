---
name: internal-standard-selection-rsdqc-ranking
description: Use when a targeted LC-MS metabolomics experiment contains multiple candidate
  internal standards and you need to assign each compound a suitable IS, using the
  lowest-RSDQC criterion on batch-corrected compound/IS ratios.
when_to_use_negative:
- When only one internal standard is available per metabolite and no ranking between
  candidates is needed — mzQuality's doAnalysis handles the single-assigned-IS ratio
  step directly.
- When the goal is overall compound-quality triage rather than IS choice — use the
  RSDqc / background-signal (e-score) classification into High Confidence, With Caution
  and Low SNR tabs instead.
- When data are raw vendor peak tables not yet integrated — mzQuality operates on
  peak-integrated data (e.g. from SCIEX OS) and does not process peaks itself.
tools:
- name: mzQuality
  role: R package that performs the batch correction, outlier flagging and the 'Suggest
    Internal Standards' step the RSDQC ranking reproduces; readData/buildExperiment/doAnalysis
    prepare the inputs
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment (>= 4.0) in which mzQuality is distributed and the RSDQC
    ranking is computed
- name: mzQualityDashboard
  role: Shiny dashboard built on mzQuality that exposes the same IS-suggestion results
    interactively; recommended for new R users
  repo: hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:5feb0e83fd04d71630c3e2d1e8ccec7f80d822a37cec47c93e4fb0113c15f869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# internal-standard-selection-rsdqc-ranking

## Summary

Selects the best internal standard for each metabolite in a targeted LC-MS dataset by computing the relative standard deviation of QC samples (RSDQC) for every compound/IS ratio after batch correction, then ranking IS candidates by lowest RSDQC, mirroring mzQuality's suggestion logic.

## When to use

Use when a targeted LC-MS metabolomics experiment contains multiple candidate internal standards and you need to assign each compound a suitable IS, using the lowest-RSDQC criterion on batch-corrected compound/IS ratios.

## When NOT to use

- When only one internal standard is available per metabolite and no ranking between candidates is needed — mzQuality's doAnalysis handles the single-assigned-IS ratio step directly.
- When the goal is overall compound-quality triage rather than IS choice — use the RSDqc / background-signal (e-score) classification into High Confidence, With Caution and Low SNR tabs instead.
- When data are raw vendor peak tables not yet integrated — mzQuality operates on peak-integrated data (e.g. from SCIEX OS) and does not process peaks itself.

## Inputs

- Tab-delimited peak-integrated metabolomics table with mandatory columns (aliquot, compound, area, type, injection_time, batch), e.g. mzQuality extdata/example.tsv
- SummarizedExperiment object built by mzQuality's buildExperiment
- Pooled study quality control (SQC) sample annotations used for RSDQC calculation

## Outputs

- Ranked table of internal standard candidates per compound, sorted ascending by RSDQC of batch-corrected compound/IS ratios
- Recommended internal standard assignment per compound (the IS with lowest RSDQC), e.g. written to is_selection_rsdqc.csv
- RSDQC values for each compound/IS combination comparable to mzQuality's 'Suggest Internal Standards' analysis step

## How to apply

Load the tab-delimited peak-integrated data (e.g. mzQuality's extdata/example.tsv, containing aliquot, compound, area, type, injection_time and batch columns) with readData and build it into a SummarizedExperiment with buildExperiment. Apply mzQuality's between-batch adjustment to compute batch-corrected ratios of each compound's peak area against each candidate internal standard. Exclude flagged potential outliers and treat any NA value as an absent data point before QC-sample calculations, per mzQuality's handling rules. For each candidate IS, calculate the relative standard deviation of QC samples (RSDQC) of the corrected ratios and rank candidates in ascending RSDQC, selecting the lowest as the recommended IS. Interpretation should respect the article's QC-variation thresholds: RSDqc <15% is high confidence, 15–30% is 'With Caution', and >30% fails the guideline threshold, so a recommended IS whose RSDQC remains above 30% signals poor compound quality rather than a good IS match.

## Related tools

- **mzQuality** (R package that performs the batch correction, outlier flagging and the 'Suggest Internal Standards' step the RSDQC ranking reproduces; readData/buildExperiment/doAnalysis prepare the inputs) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment (>= 4.0) in which mzQuality is distributed and the RSDQC ranking is computed)
- **mzQualityDashboard** (Shiny dashboard built on mzQuality that exposes the same IS-suggestion results interactively; recommended for new R users) — hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- The mzQuality-recommended internal standard for a compound corresponds to the candidate with the lowest RSDQC in the ranking output.
- RSDQC for the selected IS falls within the accepted QC-variation bands: <15% high confidence, 15–30% 'With Caution'; a value >30% exceeds the guideline threshold and should not be treated as a good IS match.
- NA values and mzQuality-flagged potential outliers are excluded from the QC-sample statistics (NA treated as absent data point, outliers 'automatically excluded from further calculations').
- Ratios are computed on batch-corrected values (assay 'ratio_corrected'), matching mzQuality's between-batch adjustment, not on raw peak areas.
- The candidate set is exhaustive over all compound/IS combinations, as in the validation task reproducing mzQuality's suggestion over all pairings.

## Limitations

- The article/README do not name a specific version of mzQuality for these calculations ('No specific version of mzQuality stated').
- The supplied example dataset (105 compounds, 584 samples) is not identified by name or accession beyond extdata/example.tsv.
- Thresholds (RSDqc 30%, e-score 40%) are user-definable in doAnalysis; the ranking inherits whatever thresholds the user set rather than fixed defaults.
- When a compound is not detected in LQC or SQC samples, mzQuality assumes it is not present and will not report it, so no IS ranking is possible.

## Evidence

- [readme] Suggest Internal Standards based on the calculated values: [evidence span withheld — anchor status: external_doc]
- [readme] RSDQC drives the decision for compounds: "It bases the decision for compounds on the combination of RSDQC, 
the background signal percentage, and the presence of the compounds in QC 
samples."
- [results] Potential outliers are automatically excluded from calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [results] NA values treated as absent data points: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [results] Threshold for RSDqc set at 30% per metabolomics QA/QC consortium: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [results] Compounds organized into three quality tabs: "Compounds are organized into three tabs based on quality and user-defined or default settings"
