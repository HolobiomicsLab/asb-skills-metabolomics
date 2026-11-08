---
name: reference-value-verification-of-model-predictions
description: Use when a published model yields concrete numeric reference values (e.g.,
  20%-change stability time points in EDTA/LiHep/serum) that a recomputation from
  source data must reproduce, or when validating that a re-implementation matches
  reported anchors.
when_to_use_negative:
- When no published reference values or anchors exist to compare against — the skill
  is a verification step, not a primary modeling method
- When the input is raw NMR spectra or unprocessed metabolite intensities rather than
  structured delay-experiment tables keyed by tube type and delay time
- When the goal is to fit new stability models for novel tube types or thresholds
  rather than confirm that a recomputation reproduces the reported anchors
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: R
  role: Environment for loading the delay-experiment tables, fitting the linear mixed
    models (cell count random slope, donor random intercept), and extracting 20%-change
    stability time points
- name: PRIMA Panel
  role: Reference implementation whose published stability time points serve as the
    verification anchors; also exposes the stability time points in its Data tab for
    cross-checking
  repo: github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:25aa5c6e1df67f9ae8cece742452ee68d15372a4f7aa948c49dd41fac715bb66
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA
  - PRIMA Panel
  - R
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Reference-value verification of model predictions

## Summary

Recompute quantitative model outputs (here, metabolite stability time points from linear mixed models of pre-/postcentrifugation delays) and verify each against published reference anchors, flagging parameters whose recomputed values diverge for expert review.

## When to use

Use when a published model yields concrete numeric reference values (e.g., 20%-change stability time points in EDTA/LiHep/serum) that a recomputation from source data must reproduce, or when validating that a re-implementation matches reported anchors.

## When NOT to use

- When no published reference values or anchors exist to compare against — the skill is a verification step, not a primary modeling method
- When the input is raw NMR spectra or unprocessed metabolite intensities rather than structured delay-experiment tables keyed by tube type and delay time
- When the goal is to fit new stability models for novel tube types or thresholds rather than confirm that a recomputation reproduces the reported anchors

## Inputs

- Original delay-experiment data tables from the PRIMA GitHub repository (github.com/funkam/PRIMA), with tube type, pre- and postcentrifugation delay times, donor identifiers, and cell counts
- Published reference anchor values (e.g., lactic acid precentrifugation 0.41 h in EDTA; postcentrifugation 0.81 h EDTA, 1.63 h LiHep, 4.3 h serum)
- Fitted linear mixed model objects or coefficients per metabolic parameter

## Outputs

- results_table.csv with recomputed stability time points per metabolic parameter, tube type (EDTA, LiHep, serum), and delay type (pre-/postcentrifugation)
- Flag list of parameters whose recomputed stability time points diverge from reported values, for expert review

## How to apply

Download the original delay-experiment data tables from github.com/funkam/PRIMA and structure them per metabolic parameter with tube type (EDTA, LiHep, serum), pre-/postcentrifugation delay times, donor identifiers, and cell counts. For each parameter, fit the linear mixed model with cell count as random slope and donor as random intercept, then extract the modeled time at which concentration changes by 20% (the 'stability time point'). Compare each recomputed value against the reported anchors: lactic acid precentrifugation 0.41 h in EDTA plasma, and postcentrifugation 0.81 h (EDTA), 1.63 h (LiHep), 4.3 h (serum), plus lactic acid/glucose ratio changes of 21–32 min. Accept matches within the published table's reported precision; write all recomputed values per parameter, tube type, and delay type to results_table.csv and explicitly flag any divergent parameter rather than silently accepting it.

## Related tools

- **R** (Environment for loading the delay-experiment tables, fitting the linear mixed models (cell count random slope, donor random intercept), and extracting 20%-change stability time points)
- **PRIMA Panel** (Reference implementation whose published stability time points serve as the verification anchors; also exposes the stability time points in its Data tab for cross-checking) — github.com/funkam/PRIMA

## Examples

```
Rscript -e 'm <- lmer(conc ~ delay + (1+delay|donor), d); tp <- 0.2/abs(fixef(m)["delay"]); write.csv(tp, "results_table.csv")'
```

## Evaluation signals

- Recomputed lactic acid precentrifugation stability time point in EDTA plasma reproduces the reported 0.41 h (~25 min) within the table's reported precision
- Recomputed postcentrifugation stability time points for lactic acid match 0.81 h (EDTA), 1.63 h (LiHep), and 4.3 h (serum)
- Recomputed lactic acid/glucose ratio change falls in the reported 21–32 min window
- results_table.csv contains one row per parameter × tube type × delay type with no missing stability time points
- Every parameter whose recomputed value diverges from the reported anchor is explicitly flagged rather than silently accepted

## Limitations

- Verification is only as good as the anchors — the underlying linear mixed models were validated in a separate cohort, so residual cohort-specific effects can propagate into the reference values
- The 20% concentration-change threshold is fixed by the published definition; other thresholds require recomputation outside this skill's verification scope
- The PRIMA repository provides no changelog or version information, so recomputed values cannot be pinned to a specific tool version

## Evidence

- [intro] linear mixed models anchor: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] lactic acid sensitivity: "some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer"
- [readme] stability time points in PRIMA Panel: "The Data tab shows different ways of highlighting the different stability time-points in minutes."
- [abstract] stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
