---
name: mixed-model-based-delay-impact-prediction
description: Use when sample cohorts carry variable pre- and postcentrifugation processing
  delays and you need to predict, per metabolite (NMR-quantified serum/plasma), which
  concentrations shift and by how much before running downstream analyses.
when_to_use_negative:
- Samples are not peripheral blood plasma/serum or were not NMR-quantified — the models
  predict changes only for these metabolic parameters under pre-/postcentrifugation
  delay
- The pre-analytical table lacks date+time stamps from which pre- and post-centrifugation
  delays can be computed — the panel's Tools tab expects date+time stamp differences
- You only need Bruker BiobankQC-style basic pass/fail output without per-metabolite
  delay-effect quantification — PRIMA was developed to enhance that basic pass/fail
  output, so a simple QC call may suffice
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: PRIMA Panel
  role: R Shiny tool that packages the linear mixed models and cohort data; used to
    explore delay effects interactively and generate per-metabolite performance reports
    from an uploaded pre-analytical table
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Implementation language of the PRIMA Panel in which the linear mixed models
    are fitted and served
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Web framework hosting the PRIMA Panel interface (Data tab, 'Single' performance-report
    tab with sliders and adjustable % thresholds), hosted on Shinyapps.io
  repo: https://github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:3de73892f08ec3a24354797d9fac08d3da987d8b13aa3a0a8de973fe0d39f59a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mixed-model-based-delay-impact-prediction

## Summary

Fit linear mixed models to predict and quantify how each NMR-quantified metabolic parameter in plasma/serum changes with pre- and postcentrifugation delays, then combine both delay types to flag metabolites undergoing minor vs. major change at a chosen tolerable-error threshold.

## When to use

Use when sample cohorts carry variable pre- and postcentrifugation processing delays and you need to predict, per metabolite (NMR-quantified serum/plasma), which concentrations shift and by how much before running downstream analyses.

## When NOT to use

- Samples are not peripheral blood plasma/serum or were not NMR-quantified — the models predict changes only for these metabolic parameters under pre-/postcentrifugation delay
- The pre-analytical table lacks date+time stamps from which pre- and post-centrifugation delays can be computed — the panel's Tools tab expects date+time stamp differences
- You only need Bruker BiobankQC-style basic pass/fail output without per-metabolite delay-effect quantification — PRIMA was developed to enhance that basic pass/fail output, so a simple QC call may suffice

## Inputs

- NMR-quantified metabolic parameter values from serum/plasma samples subjected to controlled pre- and postcentrifugation delay time courses
- Pre-analytical data table for a sample cohort (per-sample delay timestamps, SPREC-classified pre- and post-centrifugation times in minutes)
- Tolerable-error threshold (e.g. a % change threshold such as 15%) defining minor vs. major metabolite change

## Outputs

- Linear mixed models predicting concentration change per metabolic parameter as a function of pre- and postcentrifugation delay
- Per-metabolite stability time points (delay at which concentration changes by 20%)
- Performance report table flagging a minor and a major change for each metabolite at the chosen tolerable error, exportable as .csv or .HTML

## How to apply

Start from a controlled preanalytical experiment in which blood collected in various tube types is subjected to defined pre- and postcentrifugation delays, and metabolite levels are quantified by NMR spectroscopy. Use these time-course data to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays; for each metabolite also derive a 'stability time point', i.e. the delay at which the concentration changes by 20%. Model the combination of pre- and postcentrifugation delays to obtain a comprehensive overview of the preanalytical process, then validate the models on a separate cohort to establish robustness. Finally, apply the models to a user cohort's pre-analytical data table (e.g. via the PRIMA Panel 'Single' tab with slider-set pre- and post-centrifugation times and adjustable % thresholds/colours) to generate a performance report that flags a minor and a major change per metabolite over the given timeframe. Validate application by checking the report reproduces known delay distributions and expected outliers such as lactic acid.

## Related tools

- **PRIMA Panel** (R Shiny tool that packages the linear mixed models and cohort data; used to explore delay effects interactively and generate per-metabolite performance reports from an uploaded pre-analytical table) — https://github.com/funkam/PRIMA
- **R** (Implementation language of the PRIMA Panel in which the linear mixed models are fitted and served) — https://github.com/funkam/PRIMA
- **Shiny** (Web framework hosting the PRIMA Panel interface (Data tab, 'Single' performance-report tab with sliders and adjustable % thresholds), hosted on Shinyapps.io) — https://github.com/funkam/PRIMA

## Examples

```
https://funkam.shinyapps.io/PRIMA/ → upload pre-analytical data table → 'Single' tab: set pre-/post-centrifugation sliders, adjust % thresholds → download performance_report.csv
```

## Evaluation signals

- Model predictions should reproduce metabolite-level behaviour seen in the controlled delay experiment, e.g. lactic acid significantly affected by even short delays while other metabolites remain stable for longer
- For each metabolite, the derived stability time point must mark the delay where concentration changes by 20%
- Applied to a validation cohort, model outputs must remain robust (the paper reports models validated in a separate cohort)
- Performance report for a real biobank cohort (e.g. Dresden Integrated Liquid Biobank) should show most metabolites altered by around 5% at 15% tolerable error, apart from expected outliers like glycine and lactate
- Generated table must contain a minor and a major change entry per metabolite and export cleanly as .csv or .HTML from the PRIMA Panel

## Limitations

- Models are grounded in controlled pre-/postcentrifugation delay experiments on peripheral blood plasma/serum; transferability to other matrices or processing perturbations is not demonstrated
- Robustness depends on validation-cohort coverage; the paper notes a validation cohort was used to investigate robustness, implying out-of-scope cohorts need re-validation
- The PRIMA Panel README provides no version information and no changelog, so pinning tool state is difficult
- Threshold colours and % thresholds are user-adjustable, so reports are only comparable if thresholds are fixed a priori (e.g. 15% tolerable error)

## Evidence

- [abstract] mixed models generated from controlled delay time courses: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [intro] models quantified changes per metabolic parameter under prolonged delays: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [discussion] combined pre/post delay modeling: "a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples"
- [discussion] validation in separate cohort: "A validation cohort was used to investigate the robustness of these models"
- [readme] PRIMA Panel single-tab report generation with adjustable thresholds: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite in"
- [results] example application reproducing DILB findings: [evidence span withheld — anchor status: not_found]
- [intro] PRIMA scope versus BiobankQC: "PRIMA was developed to enhance this functionality, offering a broader range of information on processing delays, and a user-friendly overview tailored for biobank users."
