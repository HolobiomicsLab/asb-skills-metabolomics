---
name: blank-to-sample-intensity-ratio-analysis
description: Use when a metabolomics/lipidomics sequence includes procedural blank
  injections alongside real samples and you must separate genuine endogenous signals
  from carryover or background contaminants before further QC.
when_to_use_negative:
- Sequence contains no procedural blank injections — signal-drift, outlier, and missing-value
  QC steps of QComics apply instead.
- Features are already known internal standards or labeled surrogates whose blank
  behavior is irrelevant to the decision.
- Data are not MS-based metabolomics/lipidomics feature intensities (e.g. raw MS/MS
  spectra or non-omics data types the paper does not describe).
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: R package implementing the sequential QC protocol of which the blank/sample
    3x background-contamination criterion is a step; provides example data_df and
    RSD-based flagging logic
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment for installing and running the QComics app or scripting
    the blank-vs-sample ratio computation
  repo: github.com/ricoderks/QComics
- name: RStudio
  role: Recommended front-end for launching QComics::run_app() locally
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:32ff7cd2e4aee736a959570ecfaecd623df05afb18238c8eff93c8e25f60bff8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - RStudio
  - dplyr
  - readr
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# blank-to-sample-intensity-ratio-analysis

## Summary

Screens metabolomics features for background contamination by comparing mean intensity in real study samples against mean intensity in procedural blank injections, using the QComics 3x sample-to-blank criterion to flag rather than blindly remove features.

## When to use

Use when a metabolomics/lipidomics sequence includes procedural blank injections alongside real samples and you must separate genuine endogenous signals from carryover or background contaminants before further QC.

## When NOT to use

- Sequence contains no procedural blank injections — signal-drift, outlier, and missing-value QC steps of QComics apply instead.
- Features are already known internal standards or labeled surrogates whose blank behavior is irrelevant to the decision.
- Data are not MS-based metabolomics/lipidomics feature intensities (e.g. raw MS/MS spectra or non-omics data types the paper does not describe).

## Inputs

- Feature intensity table (data_df) with rows as molecular features and columns separating procedural blank injections from real study samples
- Procedural blank injection intensities from the sequence
- Real sample intensities from the sequence
- Optional blank replicate groups for computing blank RSD

## Outputs

- Per-feature contaminant flag (contaminant vs retained) derived from the sample/blank mean ratio ≤ 3x rule
- Summary table of blank mean, real-sample mean, sample/blank ratio, and flag decision per feature
- Recommended discard subset: flagged features with blank RSD > 15%

## How to apply

Load a feature-intensity table (data_df) whose rows are features and whose columns include both procedural blank injections and real study samples, as shipped with the QComics R package (github.com/ricoderks/QComics). For every feature compute the mean intensity across blank injections and across real samples, then apply the background-contamination criterion: flag the feature as a potential contaminant when its mean in real samples is not more than three times (≤ 3x) its mean in blanks. Per QComics policy the flagged features are marked for review rather than removed upfront; only flagged features whose blank relative standard deviation exceeds 15% (blank RSD > 15%) are recommended for outright discard. Assemble a per-feature summary table (blank mean, sample mean, sample/blank ratio, flag/retain decision) so that the contaminant screen is auditable and reportable alongside the rest of the QComics QC chain (background noise and carryover correction, drift detection, missing-value handling).

## Related tools

- **QComics** (R package implementing the sequential QC protocol of which the blank/sample 3x background-contamination criterion is a step; provides example data_df and RSD-based flagging logic) — github.com/ricoderks/QComics
- **R** (Runtime environment for installing and running the QComics app or scripting the blank-vs-sample ratio computation) — github.com/ricoderks/QComics
- **RStudio** (Recommended front-end for launching QComics::run_app() locally)

## Examples

```
remotes::install_github("ricoderks/QComics")
```

## Evaluation signals

- Every feature in the output table carries a computed sample/blank mean ratio, not just a boolean flag.
- Flagged set exactly matches an expert-reviewed application of the ≤ 3x rule to the same blank/sample data.
- Among flagged features, those recommended for removal all have blank RSD exceeding 15%.
- Retained features include any whose sample mean exceeds 3x its blank mean, regardless of RSD.
- Output is auditable: ratio and threshold logic can be re-derived from the stored mean values.

## Limitations

- The 3x cutoff and 15% blank-RSD threshold are QComics-specific quality criteria adapted from bioanalytical validation guidelines and current metabolomics QC literature, not universally standardized values.
- Features are flagged rather than removed upfront, so downstream steps must handle the flagged set explicitly.
- Missing values and truly absent data must be handled separately from this contamination screen to avoid losing relevant biological information.
- The QComics package is experimental (lifecycle badge) and its QCpool-based workflow assumes pooled QC samples analysed with Sciex Multiquant > v3.0.3 and exported as txt — blank handling must be reconciled with that pipeline.

## Evidence

- [other] QComics flags a feature as a potential contaminant when its mean value in real samples is not more than three times its mean value in procedural blanks: [evidence span withheld — anchor status: not_found]
- [other] only flagged features with blank RSD > 15% are recommended for discard: [evidence span withheld — anchor status: not_found]
- [abstract] protocol corrects for background noise and carryover as the initial QC step: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover"
- [intro] initial data exploration covers contaminant detection: "(i) initial data exploration (i.e., detection of contaminants, batch drifts, and out-of-control measurements)"
- [readme] QComics installs from the ricoderks GitHub repo: "remotes::install_github("ricoderks/QComics")"
- [intro] QComics quality criteria are adapted from bioanalytical validation guidelines and metabolomics QC literature: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
