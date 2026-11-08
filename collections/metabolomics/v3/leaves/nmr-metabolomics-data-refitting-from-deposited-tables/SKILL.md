---
name: nmr-metabolomics-data-refitting-from-deposited-tables
description: Use when deposited delay-experiment data tables must be re-analyzed to
  reproduce or extend metabolite stability time points (20% change threshold) for
  EDTA plasma, LiHep plasma, or serum from pre-/postcentrifugation delays.
when_to_use_negative:
- Input is raw NMR spectra rather than structured delay-experiment tables with tube
  type, delay times, donor IDs, and cell counts — quantification must happen first.
- Goal is a quick pass/fail QC of a single sample cohort with known delays; the PRIMA
  Panel's Performance Reports ('Single') tab generates such reports directly without
  refitting models.
- Metabolites measured on non-NMR platforms where the deposited PRIMA concentration-vs-delay
  models do not apply.
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for loading the deposited delay-experiment tables and fitting
    the linear mixed models (cell count as random slope, donor as random intercept)
    used to extract stability time points
  repo: github.com/funkam/PRIMA
- name: PRIMA Panel
  role: Source of the deposited data tables, stability time points, and pre/postcentrifugation
    delay structure being re-fit; also provides the online exploration interface
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Framework hosting the PRIMA Panel app locally or via Shinyapps.io for interactive
    exploration of the refit stability time points
  repo: github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:45380a522b887e1596e20f973a7e524ee7658babf37c9edcb5a2afc9dbfc9c63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nmr-metabolomics-data-refitting-from-deposited-tables

## Summary

Re-fits linear mixed models to deposited NMR preanalytical delay-experiment tables (EDTA plasma, LiHep plasma, serum) to recompute per-metabolite 'stability time points' — the time at which a concentration changes by 20% — and writes them to a reproducible results table.

## When to use

Use when deposited delay-experiment data tables must be re-analyzed to reproduce or extend metabolite stability time points (20% change threshold) for EDTA plasma, LiHep plasma, or serum from pre-/postcentrifugation delays.

## When NOT to use

- Input is raw NMR spectra rather than structured delay-experiment tables with tube type, delay times, donor IDs, and cell counts — quantification must happen first.
- Goal is a quick pass/fail QC of a single sample cohort with known delays; the PRIMA Panel's Performance Reports ('Single') tab generates such reports directly without refitting models.
- Metabolites measured on non-NMR platforms where the deposited PRIMA concentration-vs-delay models do not apply.

## Inputs

- Delay-experiment data tables from the PRIMA repository (github.com/funkam/PRIMA), structured per metabolic parameter with tube type (EDTA, LiHep, serum), pre-/postcentrifugation delay times, donor identifiers, and cell counts
- Reported stability time point anchors for validation (e.g., lactic acid: 0.41 h pre-EDTA; 0.81 h, 1.63 h, 4.3 h postcentrifugation)

## Outputs

- results_table.csv listing recomputed stability time points per metabolic parameter, tube type, and delay type
- List of parameters whose recomputed time points diverge from reported anchors, flagged for expert review

## How to apply

Download the original delay-experiment data tables from the PRIMA GitHub repository (github.com/funkam/PRIMA) and load them into R, structuring each table per metabolic parameter with tube type (EDTA, LiHep, serum), pre- and postcentrifugation delay times, donor identifiers, and cell counts. For each metabolic parameter, fit a linear mixed model predicting concentration over prolonged processing delay, with cell count as random slope and donor as random intercept, mirroring the article's modeling strategy. Extract, per parameter and delay type (pre- vs postcentrifugation), the modeled time at which the concentration changes by 20% — the stability time point — reporting values at the table's precision in minutes/hours. Validate the refit against known anchors: lactic acid precentrifugation 0.41 h in EDTA (≈25 min) and postcentrifugation 0.81 h (EDTA), 1.63 h (LiHep), 4.3 h (serum), plus the lactic acid/glucose ratio changing in 21–32 min. Write all recomputed stability time points per parameter, tube type, and delay type to results_table.csv, flagging any parameter whose recomputed value diverges from the reported anchor for expert review.

## Related tools

- **R** (Environment for loading the deposited delay-experiment tables and fitting the linear mixed models (cell count as random slope, donor as random intercept) used to extract stability time points) — github.com/funkam/PRIMA
- **PRIMA Panel** (Source of the deposited data tables, stability time points, and pre/postcentrifugation delay structure being re-fit; also provides the online exploration interface) — github.com/funkam/PRIMA
- **Shiny** (Framework hosting the PRIMA Panel app locally or via Shinyapps.io for interactive exploration of the refit stability time points) — github.com/funkam/PRIMA

## Examples

```
shiny::runGitHub("PRIMA", "funkam")
```

## Evaluation signals

- Recomputed lactic acid precentrifugation stability time point in EDTA plasma matches the 0.41 h anchor (≈25 min) at the table's reported precision
- Recomputed lactic acid postcentrifugation stability time points match 0.81 h (EDTA), 1.63 h (LiHep), and 4.3 h (serum)
- Lactic acid/glucose ratio change falls within the reported 21–32 min range
- results_table.csv contains one row per metabolic parameter × tube type (EDTA, LiHep, serum) × delay type (pre-/postcentrifugation)
- Any parameter whose recomputed time point diverges from the reported value is explicitly flagged for expert review

## Limitations

- The PRIMA repository provides no changelog and no version information for the PRIMA Panel, so the exact data/model version underlying a refit is hard to pin down.
- Stability time points are model-based predictions from linear mixed models; their validity depends on the deposited delay-experiment design and was assessed via a separate validation cohort, whose robustness limits are not explicitly discussed in the provided context.
- Bibliographic and software metadata (version numbers) are absent from the methods section, complicating exact reproducibility checks.

## Evidence

- [abstract] lmm_prediction: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [abstract] lactic_acid_sensitivity: "some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer"
- [intro] modeling_rationale: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [readme] stability_timepoint_display: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
