---
name: sprec-delay-classification
description: Use when a biobank cohort dataset carries date/time stamps and SPREC
  classifications for pre- and postcentrifugation delays, and you need to stratify
  samples by delay category for NMR metabolomics QC.
when_to_use_negative:
- The input already contains the SPREC-classified delay tables and you only need a
  pass/fail quality verdict — Bruker's BiobankQC or the PRIMA 'Single' tab report
  is the more direct tool.
- The samples are urine or tissue rather than peripheral blood plasma/serum — the
  PRIMA Panel models processing delays for plasma/serum NMR metabolomics, and delay-stratum
  behaviour may not transfer.
- No date+time stamps or SPREC codes are available, so delays cannot be computed or
  classified.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool whose Data tab performs the SPREC-sorted delay classification
    and visualization; its Tools tab can also derive pre-/post-centrifugation times
    from date+time stamp differences
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Underlying runtime for the Shiny app; used if the analyst wants to reproduce
    the split/classification locally before loading results back into the panel
- name: Shiny
  role: Web framework hosting the interactive Data tab where the classification and
    lollipop/higlighted delay views are rendered
  repo: https://funkam.shinyapps.io/PRIMA/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:a0505775c1eb1d1872add2dc908a1e5bd78ed5a1cf5575e308ed7b9b1cea576f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sprec-delay-classification

## Summary

Use the PRIMA Panel's Data tab to split pre- and postcentrifugation delay measurements into SPREC-classified delay distributions, visualized as stability time-point highlights or lollipop plots, before further metabolomics quality assessment.

## When to use

Use when a biobank cohort dataset carries date/time stamps and SPREC classifications for pre- and postcentrifugation delays, and you need to stratify samples by delay category for NMR metabolomics QC.

## When NOT to use

- The input already contains the SPREC-classified delay tables and you only need a pass/fail quality verdict — Bruker's BiobankQC or the PRIMA 'Single' tab report is the more direct tool.
- The samples are urine or tissue rather than peripheral blood plasma/serum — the PRIMA Panel models processing delays for plasma/serum NMR metabolomics, and delay-stratum behaviour may not transfer.
- No date+time stamps or SPREC codes are available, so delays cannot be computed or classified.

## Inputs

- Data table with pre-analytical information (per-sample date+time stamps for phlebotomy and centrifugation, SPREC classification, tube type)
- Stability time-point definitions in minutes per metabolite (as used by the PRIMA Panel)

## Outputs

- Classified delay distribution plots (stability time-points highlighted in minutes, sorted by SPREC classification)
- Lollipop plots as an alternative delay-distribution visualization
- Split pre-centrifugation and post-centrifugation delay tables ready for performance-report generation

## How to apply

Load the cohort's pre-analytical data table into the PRIMA Panel (R Shiny) and open the Data tab, where delay time-points are sorted according to their SPREC classification. Split the samples into the two delay types (pre- and postcentrifugation) using the tool's built-in splitting; the README notes the data is split according to these two delays. Visualize the delay distributions either as highlighted stability time-points (in minutes) or as lollipop plots, per the alternative presentation mode. These classified delay distributions then feed downstream performance-report generation in the PRIMA Panel's 'Single' tab, where pre- and postcentrifugation times are set with sliders and per-metabolite minor/major change thresholds are computed. Rationale: the classification anchors the subsequent linear mixed-model predictions to realistic delay strata observed in the cohort, e.g. the DILB cohort of 765 serum samples whose delays fell mostly in SPREC categories A1/A/C pre- and B/D/F post-centrifugation.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool whose Data tab performs the SPREC-sorted delay classification and visualization; its Tools tab can also derive pre-/post-centrifugation times from date+time stamp differences) — https://github.com/funkam/PRIMA
- **R** (Underlying runtime for the Shiny app; used if the analyst wants to reproduce the split/classification locally before loading results back into the panel)
- **Shiny** (Web framework hosting the interactive Data tab where the classification and lollipop/higlighted delay views are rendered) — https://funkam.shinyapps.io/PRIMA/

## Examples

```
https://funkam.shinyapps.io/PRIMA/ — open the Data tab, upload the pre-analytical CSV, and review the SPREC-sorted pre-/post-centrifugation delay distributions.
```

## Evaluation signals

- The delay distribution should stratify into SPREC categories consistent with the cohort's known distribution — for the DILB use-case, pre-centrifugation was ~59.7% A / 4.3% A1 / 28.9% C / 6.8% E and post-centrifugation ~48.6% B / 34.9% D / 16.5% F.
- Every sample must be assigned to exactly one pre-centrifugation and one post-centrifugation delay stratum; no unclassified samples should remain.
- Stability time-points displayed must be in minutes, matching the PRIMA Panel's internal units.
- The split must produce two disjoint groups (pre- vs post-centrifugation) whose sizes sum to the total cohort size.

## Limitations

- The SPREC classification only stratifies delays; it does not itself quantify the metabolite-level impact — that requires the downstream linear mixed models and the 'Single' tab's minor/major change thresholds.
- The panel is designed for peripheral blood (plasma/serum) NMR metabolomics; delay distributions derived for other specimen types are not validated.
- If raw date+time stamps rather than SPREC codes are the only input, delays must first be computed with the panel's Tools tab before classification.

## Evidence

- [readme] Data tab SPREC sorting and stability time-points: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] Split by pre-/post-centrifugation delays: "The data is split according the two different delays (pre- and post-centrifugation"
- [readme] Lollipop plot alternative: "In addition, there is an alternative way of presenting the date in form of lollipop plots."
- [abstract] PRIMA Panel purpose for delay effects: "the authors developed the PRIMA Panel, an open-source R Shiny tool"
- [abstract] DILB cohort used to validate SPREC delay stratification: "The PRIMA Panel was tested using samples from the Dresden Integrated Liquid Biobank"
