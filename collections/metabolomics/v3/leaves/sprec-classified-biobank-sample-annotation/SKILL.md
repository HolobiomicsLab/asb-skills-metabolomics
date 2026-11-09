---
name: sprec-classified-biobank-sample-annotation
description: Use when a metabolomics dataset contains peripheral-blood samples with
  variable pre- and post-centrifugation delays and you need sample-type-aware stability
  annotation (EDTA plasma, serum, LiHep plasma) sorted by SPREC classification.
when_to_use_negative:
- Samples are not peripheral blood plasma/serum (e.g., urine or tissue homogenates)
  — the panel's models and stability time points were derived for blood-derived matrices
  only.
- You only need Bruker BiobankQC's pass/fail output and do not need per-parameter
  stability annotation or preanalytical delay exploration.
- Sample cohorts lack date/time stamps or preanalytical metadata, so pre- and post-centrifugation
  times cannot be computed or verified.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0736
tools:
- name: PRIMA Panel
  role: Interactive R Shiny tool whose Data tab renders stability time points sorted
    by SPREC classification and split by pre-/post-centrifugation delay, and whose
    Tools tab converts date+time stamp differences into pre-/post-centrifugation times
    for annotating sample cohorts
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Underlying language in which the annotation table and stability time point
    computations are implemented for the PRIMA Panel
  repo: https://github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:84738de78b346564a45b83d016e28f461a3535177ee775a838a21978bae19cf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# sprec-classified-biobank-sample-annotation

## Summary

Annotate biobank-derived metabolomics samples with SPREC-classified pre- and post-centrifugation delay times so stability time points can be attached per sample type (EDTA plasma, serum, LiHep plasma) and delay type before downstream thresholding.

## When to use

Use when a metabolomics dataset contains peripheral-blood samples with variable pre- and post-centrifugation delays and you need sample-type-aware stability annotation (EDTA plasma, serum, LiHep plasma) sorted by SPREC classification.

## When NOT to use

- Samples are not peripheral blood plasma/serum (e.g., urine or tissue homogenates) — the panel's models and stability time points were derived for blood-derived matrices only.
- You only need Bruker BiobankQC's pass/fail output and do not need per-parameter stability annotation or preanalytical delay exploration.
- Sample cohorts lack date/time stamps or preanalytical metadata, so pre- and post-centrifugation times cannot be computed or verified.

## Inputs

- Data table with pre-analytical information for a sample cohort (sample type, collection/centrifugation/measurement date+time stamps)
- Metabolic parameters with stability time points expressed in minutes
- SPREC classification labels for each biospecimen (EDTA plasma, serum, LiHep plasma)

## Outputs

- Stability time point table in which each metabolic parameter row carries sample type (EDTA plasma / serum / LiHep), delay type (pre- vs postcentrifugation), and SPREC-sorted ordering
- Computed pre- and post-centrifugation times (in minutes) derived from date+time stamp differences
- Optionally, lollipop-plot visualizations of stability time points split by pre- and post-centrifugation delay

## How to apply

Build a table in which each metabolic parameter's stability time point (expressed in minutes) is annotated with its sample type (EDTA plasma, serum, LiHep plasma) and delay type (pre- versus postcentrifugation), then sort the rows by their SPREC classification so stability points are grouped by biospecimen classification. Split the data into the two delay categories before further analysis — the PRIMA Panel's Data tab uses exactly this SPREC-based sorting and pre-/post-centrifugation split, optionally visualized as lollipop plots. When preanalytical information for a whole cohort is available, upload a data table containing the sample types and the date/time stamps of collection, centrifugation, and measurement; the panel's Tools module converts date+time stamp differences into pre- and post-centrifugation times, which are then attached to each sample. The rationale is that stability time points differ strongly between sample types (e.g., 26 metabolic parameters have a precentrifugation stability time point within the first 4 h for both EDTA plasma and serum, but only 15 for LiHep plasma), so annotation without SPREC-aware sample typing would mask these differences.

## Related tools

- **PRIMA Panel** (Interactive R Shiny tool whose Data tab renders stability time points sorted by SPREC classification and split by pre-/post-centrifugation delay, and whose Tools tab converts date+time stamp differences into pre-/post-centrifugation times for annotating sample cohorts) — https://github.com/funkam/PRIMA
- **R** (Underlying language in which the annotation table and stability time point computations are implemented for the PRIMA Panel) — https://github.com/funkam/PRIMA

## Examples

```
Rscript -e "shiny::runGitHub('funkam/PRIMA')"  # Data tab: SPREC-sorted stability time points, split by pre-/post-centrifugation delay
```

## Evaluation signals

- Every metabolic parameter row must carry a non-empty sample type label from {EDTA plasma, serum, LiHep plasma} and a delay type from {precentrifugation, postcentrifugation}.
- Rows are ordered by SPREC classification (not alphabetically or by parameter name) as in the PRIMA Panel Data tab.
- Pre- and post-centrifugation times computed from date+time stamp differences must be positive and non-decreasing along the processing chain (collection → centrifugation → measurement).
- After converting minutes to hours, counts of parameters with stability time point ≤ 240 min (4 h) should reproduce the reported values: precentrifugation 26 (EDTA plasma), 26 (serum), 15 (LiHep); postcentrifugation 26 (EDTA plasma), 23 (serum), 8 (LiHep).
- Lollipop-plot alternative rendering must contain the same stability time points and splits as the tabular view (no data loss between representations).

## Limitations

- Stability time points are defined at a 20% change threshold of metabolite concentration; delays are annotated to this single threshold, so sub-20% drifts are not captured.
- Annotation quality depends entirely on complete and correct date+time stamps and accurate SPREC classification of each tube; missing or wrong stamps silently distort the computed delays.
- The tool investigates effects of processing delays on peripheral blood (plasma/serum) samples; sample types outside this scope are not supported.

## Evidence

- [readme] stability time point sorted by SPREC: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] delay split pre/post: "The data is split according the two different delays (pre- and post-centrifugation"
- [readme] preanalytical table upload: "Here a data table with pre-analytical information can be uploaded"
- [readme] tools computes delay times: "Currently it consists of a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
- [readme] purpose of panel: "The PRIMA-Panel is a tool to investigate the effect of processing delays on metabolic parameters in samples of peripheral blood (plasma / serum)."
- [intro] delay types investigated: "we investigated the detailed effects of prolonged pre- and postcentrifugation times on metabolic parameters using NMR Spectroscopy"
- [abstract] stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [other] sample-type-dependent counts within 4h: [evidence span withheld — anchor status: not_found]
- [discussion] PRIMA panel purpose: "these models and data were combined to create the PRIMA Panel, which allows the exploration of the data and provides output in form of performance reports"
