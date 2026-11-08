---
name: metabolomics-feature-intensity-table-processing
description: Use when a metabolomics feature intensity table contains blank injections
  and features flagged as 'potential contaminants' (mean in real samples not exceeding
  three times the blank mean), and you must decide which contaminant peaks to discard
  before data analysis.
when_to_use_negative:
- The input is raw instrument output (e.g., unprocessed Sciex Multiquant txt exports)
  rather than a processed feature intensity table.
- The dataset contains no blank injections or no features previously flagged as 'potential
  contaminants' via the three-times-blank-mean criterion.
- You are applying the 15% RSD cutoff to non-contaminant endogenous features — the
  threshold as stated applies to contaminant features of potential interest, not general
  feature filtering.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: QComics
  role: QC protocol whose blank-variability guidance defines the 15% RSD discard threshold
    for contaminant features; its repository provides the example blank-injection
    table used as input
  repo: github.com/ricoderks/QComics
- name: R
  role: Runtime environment in which the filter script is executed (via RStudio)
- name: dplyr
  role: mutate/case_when for RSD-based classification and bind_rows for combining
    labeled 'discarded'/'retained' groups
- name: readr
  role: read_csv to load the blank-injection feature table and write_csv to export
    blank_variability_filter_results.csv
- name: stats
  role: sd() and mean() to compute per-feature RSD as (SD / mean) x 100 across blank
    injections
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:e23e78f65eca04e15606b8fcba12333cd025d4e9a874a1aa48947b039daf5276
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QComics
  - R
  - dplyr
  - readr
  - stats
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomics-feature-intensity-table-processing

## Summary

Implements the QComics blank-variability filter for 'potential contaminant' molecular features: compute the per-feature RSD across blank injections and discard only those features whose blank RSD exceeds 15%, retaining low-variability contaminants for downstream group comparisons.

## When to use

Use when a metabolomics feature intensity table contains blank injections and features flagged as 'potential contaminants' (mean in real samples not exceeding three times the blank mean), and you must decide which contaminant peaks to discard before data analysis.

## When NOT to use

- The input is raw instrument output (e.g., unprocessed Sciex Multiquant txt exports) rather than a processed feature intensity table.
- The dataset contains no blank injections or no features previously flagged as 'potential contaminants' via the three-times-blank-mean criterion.
- You are applying the 15% RSD cutoff to non-contaminant endogenous features — the threshold as stated applies to contaminant features of potential interest, not general feature filtering.

## Inputs

- Blank-injection feature intensity table (CSV as provided with the QComics repository example data)
- Feature identifier column plus per-blank-injection intensity columns
- Prior classification of features as 'potential contaminants' (mean in real samples not exceeding three times the blank mean)

## Outputs

- blank_variability_filter_results.csv — tidy CSV with feature identifier, computed blank RSD (%), and filter decision ('discarded' or 'retained') per feature

## How to apply

Load the blank-injection feature intensity table (e.g., the example data shipped with the QComics repository) using readr. Compute the per-feature RSD across blank injections as (SD / mean) x 100 with the stats functions sd() and mean(). Classify each feature with dplyr::mutate/case_when as 'discarded' when its blank RSD is greater than 15%, otherwise 'retained' — the rationale is that contaminant peaks are deliberately not removed prior to data analysis, and low blank-related variability is not expected to differentially affect group comparisons or statistical testing. Bind the two labeled groups into a single tidy table with dplyr::bind_rows, keeping the feature identifier, computed RSD, and filter decision columns. Export the labeled table with readr::write_csv as blank_variability_filter_results.csv.

## Related tools

- **QComics** (QC protocol whose blank-variability guidance defines the 15% RSD discard threshold for contaminant features; its repository provides the example blank-injection table used as input) — github.com/ricoderks/QComics
- **R** (Runtime environment in which the filter script is executed (via RStudio))
- **dplyr** (mutate/case_when for RSD-based classification and bind_rows for combining labeled 'discarded'/'retained' groups)
- **readr** (read_csv to load the blank-injection feature table and write_csv to export blank_variability_filter_results.csv)
- **stats** (sd() and mean() to compute per-feature RSD as (SD / mean) x 100 across blank injections)

## Examples

```
library(dplyr); blanks %>% mutate(rsd_pct = sd / mean * 100, decision = case_when(rsd_pct > 15 ~ "discarded", TRUE ~ "retained")) %>% write_csv("blank_variability_filter_results.csv")
```

## Evaluation signals

- The R script executes without errors and blank_variability_filter_results.csv exists on disk.
- Threshold logic holds: every row labeled 'discarded' has RSD > 15% and every row labeled 'retained' has RSD <= 15%.
- Total row count of the output table equals the number of input features (no features lost or duplicated by the bind_rows step).
- Computed RSD values are consistent with (sd(x) / mean(x)) * 100 recomputed independently over the blank-injection columns.
- Only features previously flagged as 'potential contaminants' (mean in real samples not exceeding three times the blank mean) appear in the filtered table.

## Limitations

- The 15% RSD cutoff is specific to contaminant features of potential interest; QComics does not recommend removing these peaks prior to data analysis, so the filter decision must be carried forward rather than used to drop rows from the intensity table.
- The QComics package is marked experimental and expects QCpool samples exported from Sciex Multiquant (> v3.0.3) in txt format; data from other acquisition/export pipelines may require adaptation.
- The article's tabulated RSD estimations (Figures 2–4) were not available in the provided material, so exact per-step reporting formats cannot be reproduced from context alone.

## Evidence

- [other] QComics recommends discarding only those whose RSD across blank injections exceeds 15%, since low blank-related variability is not expected to differentially affect group comparisons or statistical testing.: [evidence span withheld — anchor status: not_found]
- [abstract] The protocol corrects background noise and carryover as its first sequential QC step, of which blank-variability filtering is a part.: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover, (ii) detect signal drifts and “out-of-control” observations"
- [other] Contaminant features are flagged via the three-times-blank-mean rule before the RSD filter is applied.: [evidence span withheld — anchor status: not_found]
- [readme] The QComics package scope covers quality overview of metabolomics and lipidomics studies using regularly measured pooled QC samples.: "The goal of the `QComics` package is to have a quick overview of the quality of a metabolomics or lipidomics study"
