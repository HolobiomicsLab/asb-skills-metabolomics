---
name: missing-value-versus-absent-data-handling
description: Use when a targeted MS-based metabolomics or lipidomics QCpool data set
  (Sciex MultiQuant txt exports) contains features that are absent in some QCpool
  injections, and you must decide whether to impute, flag, or drop them without losing
  relevant biological information.
when_to_use_negative:
- When the input is already a cleaned feature table with no missing/absent entries
  and QCpool data are not involved — the QComics workflow is specific to QCpool-based
  QC of targeted MS metabolomics data.
- When analysing untargeted data where absence cannot be reliably distinguished from
  noise, as QComics was designed for targeted (Sciex MultiQuant) outputs.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: Shiny app that executes the sequential multistep QC workflow, including the
    missing-data handling step
  repo: github.com/ricoderks/QComics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:7cc1d1bd0a16971b4d889300f2d444efdccb1cd9c7f3cd849727ca8febfee294
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# missing-value-versus-absent-data-handling

## Summary

A QComics QC workflow stage that distinguishes 'missing' values (features below detection in a run but potentially detectable elsewhere) from 'truly absent' data (features genuinely not present in the sample matrix), so that imputation or exclusion decisions do not discard biologically meaningful information.

## When to use

Use when a targeted MS-based metabolomics or lipidomics QCpool data set (Sciex MultiQuant txt exports) contains features that are absent in some QCpool injections, and you must decide whether to impute, flag, or drop them without losing relevant biological information.

## When NOT to use

- When the input is already a cleaned feature table with no missing/absent entries and QCpool data are not involved — the QComics workflow is specific to QCpool-based QC of targeted MS metabolomics data.
- When analysing untargeted data where absence cannot be reliably distinguished from noise, as QComics was designed for targeted (Sciex MultiQuant) outputs.

## Inputs

- QCpool txt files exported from Sciex MultiQuant (> v3.0.3)
- A data matrix of targeted metabolomics/lipidomics features measured at regular intervals across one or more sequences

## Outputs

- A feature classification separating 'missing' from 'truly absent' values
- An updated feature table that excludes truly absent data while retaining imputable missing values for downstream RSD estimation

## How to apply

Load QCpool txt files exported from Sciex MultiQuant (> v3.0.3) into the QComics Shiny app and run the workflow through the earlier steps (background noise/carryover correction, drift/out-of-control detection) before handling missing data. In the missing-data step, separately classify features as (i) 'missing' — signals below the detection threshold for a subset of QCpool injections but otherwise present — and (ii) 'truly absent' — signals not detected at all, indicating the feature is genuinely outside the sample matrix. Treat these two categories differently: missing values can be handled by imputation or retained for later RSD evaluation, whereas truly absent data should be excluded from further statistical analysis because they carry no informative variance. Rationale: collapsing both categories into a single 'remove all NAs' rule loses relevant biological information, a pitfall the authors highlight as often neglected in existing QC protocols.

## Related tools

- **QComics** (Shiny app that executes the sequential multistep QC workflow, including the missing-data handling step) — github.com/ricoderks/QComics

## Examples

```
QComics::run_app()
```

## Evaluation signals

- The output feature table contains no entries for features classified as truly absent (i.e., zero detection across all QCpool injections).
- Features flagged as 'missing' (some QCpool injections below detection) remain in the table and are candidates for imputation or RSD evaluation.
- The separation is applied only after background noise/carryover correction and drift/out-of-control detection have been completed in the QComics sequence.
- The RSD table produced downstream reflects only features that survived the missing/absent classification, not the full raw feature list.

## Limitations

- QComics is marked experimental (lifecycle badge) and requires QCpool data analysed with Sciex MultiQuant > v3.0.3; other acquisition software outputs are not covered.
- The protocol was designed for MS-based metabolomics/lipidomics; while authors note it can manage other omics and MS data, the missing/absent classification thresholds are validated mainly for targeted metabolomics.
- No quantitative benchmark or validation metrics for the missing/absent rule are reported in the article.

## Evidence

- [abstract] separately handling missing values and truly absent data to avoid losing relevant biological information: "such as the need of separately handling missing values and truly absent data to avoid losing relevant biological information"
- [abstract] iii) deal with missing data: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations, (iii) deal with missing data,"
- [readme] qcpool samples measured in regular intervals: "a pooled sample (QCpool) needs to measured in regular intervals during one or more sequences. These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt`"
