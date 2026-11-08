---
name: blank-injection-rsd-calculation
description: Use when a metabolomics feature table includes blank injections and features
  flagged as potential contaminants (sample mean not exceeding 3× blank mean) must
  be triaged for discard versus retention before statistical analysis.
when_to_use_negative:
- The input has no blank injections (only pooled QC samples and study samples) — signal
  drift assessment, not blank-based contaminant triage, is the applicable QComics
  step
- Features are true absent/missing-value cases handled by the separate QComics missing-data
  step rather than contaminant-flagged peaks
- A feature is an unambiguous background contaminant to be removed outright; the 15%
  RSD rule applies only to contaminant features of potential interest that are retained
  into analysis
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QComics
  role: QC protocol that defines the contaminant flagging and the 15% blank-RSD discard
    criterion applied by this skill
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Execution environment (run via RStudio or shiny server) for the RSD computation
    and filtering script
- name: dplyr
  role: mutate/case_when for threshold classification and bind_rows for assembling
    the labeled tidy table
- name: readr
  role: Reads the blank-injection feature intensity table and writes blank_variability_filter_results.csv
- name: stats
  role: sd() and mean() used to compute per-feature RSD across blank injections
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:b0e4975e6d39408b6064b9ee241d7f7f0c2525df9d005b22cacf925e9663050c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - dplyr
  - readr
  - stats
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# blank-injection-rsd-calculation

## Summary

Computes per-feature relative standard deviation (RSD = SD/mean × 100) across blank injections to decide whether 'potential contaminant' features (mean in real samples ≤ 3× blank mean) should be discarded or retained under the QComics protocol, using a 15% RSD cutoff.

## When to use

Use when a metabolomics feature table includes blank injections and features flagged as potential contaminants (sample mean not exceeding 3× blank mean) must be triaged for discard versus retention before statistical analysis.

## When NOT to use

- The input has no blank injections (only pooled QC samples and study samples) — signal drift assessment, not blank-based contaminant triage, is the applicable QComics step
- Features are true absent/missing-value cases handled by the separate QComics missing-data step rather than contaminant-flagged peaks
- A feature is an unambiguous background contaminant to be removed outright; the 15% RSD rule applies only to contaminant features of potential interest that are retained into analysis

## Inputs

- Blank-injection feature intensity table (per-feature signal across replicate blank injections), e.g., example data shipped with the QComics repository
- Pre-computed contaminant flags (features whose mean in real samples does not exceed three times the blank mean), per the QComics initial data exploration step

## Outputs

- Per-feature RSD (%) across blank injections
- Labeled tidy table with feature identifier, computed RSD, and filter decision (discarded/retained), e.g., blank_variability_filter_results.csv

## How to apply

Load the blank-injection feature intensity table (e.g., with readr into R) and compute the per-feature RSD across blank injections as (SD / mean) × 100 using stats::sd() and mean(). Classify each feature with dplyr::mutate/case_when as 'discarded' when its blank RSD exceeds 15%, otherwise 'retained'. The rationale from QComics is that contaminant peaks are not removed prior to data analysis; among potential contaminants of interest, only those with high blank-related variability (RSD > 15%) are discarded, because low blank variability is not expected to differentially affect group comparisons or statistical testing. Bind the labeled groups into a single tidy table with dplyr::bind_rows, keeping feature identifier, computed RSD, and the filter decision, then export with readr::write_csv (e.g., blank_variability_filter_results.csv). Validate that every discarded row has RSD > 15% and every retained row has RSD ≤ 15%.

## Related tools

- **QComics** (QC protocol that defines the contaminant flagging and the 15% blank-RSD discard criterion applied by this skill) — https://github.com/ricoderks/QComics
- **R** (Execution environment (run via RStudio or shiny server) for the RSD computation and filtering script)
- **dplyr** (mutate/case_when for threshold classification and bind_rows for assembling the labeled tidy table)
- **readr** (Reads the blank-injection feature intensity table and writes blank_variability_filter_results.csv)
- **stats** (sd() and mean() used to compute per-feature RSD across blank injections)

## Examples

```
Rscript -e 'library(dplyr); blanks <- readr::read_csv("blank_injections.csv"); res <- blanks %>% mutate(RSD = sd/mean*100, decision = case_when(RSD > 15 ~ "discarded", TRUE ~ "retained")); readr::write_csv(res, "blank_variability_filter_results.csv")'
```

## Evaluation signals

- Threshold logic invariant: every row labeled 'discarded' has blank RSD > 15% and every 'retained' row has RSD ≤ 15%
- The output CSV (blank_variability_filter_results.csv) exists and contains one row per input feature with feature identifier, RSD, and decision columns
- RSD values recomputed independently (SD/mean × 100 per feature across blanks) match the script's values
- Only features flagged as potential contaminants (sample mean ≤ 3× blank mean) are subject to the RSD-based decision; non-contaminant features are unaffected

## Limitations

- QComics is marked experimental in the repository README and is under active development, so behavior may change
- The criterion is specific to contaminant features that are not removed prior to analysis; it does not define handling for missing values, signal drift, or outliers, which are separate QComics steps
- The 15% cutoff was adapted from bioanalytical method validation guidelines and existing QC literature; re-derive thresholds if data characteristics differ substantially

## Evidence

- [intro] 15% RSD discard criterion: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [intro] contaminant detection in initial exploration: "initial data exploration (i.e., detection of contaminants, batch drifts, and out-of-control measurements)"
- [readme] installation command: "remotes::install_github("ricoderks/QComics")"
- [methods] R implementation context: "From Rstudio run the app as"
