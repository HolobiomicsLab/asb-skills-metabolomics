---
name: pk-ofv-computation
description: Use when mapbayr and NONMEM yield discordant eta estimates for the same
  individual (e.g. models estimating lag time or infusion duration, or with large
  IIV) and you need to determine which tool found the more likely etas by comparing
  OFVi,MBR against a NONMEM-like OFVi recomputed from NONMEM's etas.
when_to_use_negative:
- The mapbayr and NONMEM eta estimates are concordant for the individual — OFV comparison
  is only meaningful for discordant cases
- The analysis goal is to validate agreement on PK outcomes (CL, AUC) rather than
  to adjudicate which tool's eta estimates are more likely
- The model has no posterior etas available from NONMEM to serve as the basis for
  the recomputed OFVi,NM-like
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: mapbayr
  role: Provides mapbayest() for MAP-BE and the compute_ofv machinery (mrgsolve model,
    omega inverse, sigma, observation/dose compartment flags, log-transformation flag)
    used to recompute OFVi from external eta estimates
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Implements the population PK model and solves the ODEs needed to predict concentrations
    and evaluate the OFV
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Source of the posterior eta estimates whose OFVi,NM-like value is recomputed
    for the discordant-case comparison
- name: R
  role: Environment for running mapbayr/mrgsolve computations, tallying the discordant
    fraction, and rendering the ggplot2 OFV-comparison figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:7a5cb48271a01945347937d766f9520b60aa659940cfc00b757984aab2a2b230
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FelicienLL/mapbayr
  repo_url: https://github.com/FelicienLL/mapbayr
  tools:
  - NONMEM
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pk-ofv-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recompute individual objective function values (OFVi) from one tool's posterior eta estimates inside mapbayr's compute_ofv machinery (mrgsolve model, omega inverse, sigma, compartment flags, log-transformation flag), so that mapbayr and NONMEM estimates can be compared on a common OFV scale in discordant cases.

## When to use

Use when mapbayr and NONMEM yield discordant eta estimates for the same individual (e.g. models estimating lag time or infusion duration, or with large IIV) and you need to determine which tool found the more likely etas by comparing OFVi,MBR against a NONMEM-like OFVi recomputed from NONMEM's etas.

## When NOT to use

- The mapbayr and NONMEM eta estimates are concordant for the individual — OFV comparison is only meaningful for discordant cases
- The analysis goal is to validate agreement on PK outcomes (CL, AUC) rather than to adjudicate which tool's eta estimates are more likely
- The model has no posterior etas available from NONMEM to serve as the basis for the recomputed OFVi,NM-like

## Inputs

- mapbayr estimation outputs (.rds files with posterior etas and OFVi,MBR) from the mapbayr-CPTPSP-2021 repository
- NONMEM estimation outputs and posterior eta estimates for discordant individuals
- Population PK model definitions: mrgsolve model code (.cpp), NONMEM control streams (.mod), omega matrix, sigma, residual-error and log-transformation settings
- Simulated or observed PK concentration datasets in NM-TRAN format (.csv)

## Outputs

- Per-patient OFV comparison table (ofv_comparison.csv) with OFVi,MBR vs recomputed OFVi,NM-like for each discordant patient
- OFV-comparison figure (ofv_comparison.png, ggplot2 scatter of mapbayr vs NONMEM-like OFV)
- Fraction of discordant patients with lower OFVi,MBR (reported ~61% of 371 patients in Runs 3, 4, 6 rich sampling)

## How to apply

Load the mapbayr and NONMEM estimation results (e.g. from the mapbayr-CPTPSP-2021 repository: simulated .csv profiles, .cpp mrgsolve models, .mod NONMEM controls, .rds mapbayr outputs) and select the discordant individuals whose eta estimates differ between the two tools. Extract NONMEM's posterior eta estimates together with the matching model definition (omega matrix, sigma, residual-error model, and log-transformation settings). Recompute OFVi,NM-like from the NONMEM etas using mapbayr's compute_ofv machinery, passing the mrgsolve model, omega inverse, sigma, observation/dose compartment flags, and the log-transformation flag. For each discordant patient (e.g. the 371 discordant patients in Runs 3, 4, and 6 with rich sampling), compare the recomputed OFVi,NM-like against OFVi,MBR and tally the fraction where OFVi,MBR is lower. In the source study, OFVi,MBR was lower in 225 of 371 patients (61%), supporting that mapbayr often found the more likely eta estimates.

## Related tools

- **mapbayr** (Provides mapbayest() for MAP-BE and the compute_ofv machinery (mrgsolve model, omega inverse, sigma, observation/dose compartment flags, log-transformation flag) used to recompute OFVi from external eta estimates) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Implements the population PK model and solves the ODEs needed to predict concentrations and evaluate the OFV) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Source of the posterior eta estimates whose OFVi,NM-like value is recomputed for the discordant-case comparison)
- **R** (Environment for running mapbayr/mrgsolve computations, tallying the discordant fraction, and rendering the ggplot2 OFV-comparison figure)

## Examples

```
my_est <- mapbayest(my_model, data = my_data); get_eta(my_est)
```

## Evaluation signals

- The computed fraction of discordant patients with lower OFVi,MBR reproduces the reported result (approximately 61% of the 371 discordant individuals in Runs 3, 4, and 6 with rich sampling)
- The recomputed OFVi,NM-like uses the same omega inverse, sigma, observation/dose compartment flags, and log-transformation flag as the original model definition — mismatch in any flag invalidates the comparison
- Concordant individuals (98% of test-model estimations) should show near-identical OFVi between tools; large OFV differences should only appear in identified discordant cases
- ofv_comparison.csv contains one row per discordant patient with both OFVi values, and the row count matches the expected discordant count (371 for Runs 3, 4, 6 rich sampling)

## Limitations

- The comparison rests on the authors' own computation of OFVi,NM-like being correct ('provided that our own computation of OFVi,NM-like is not false')
- Discordances arise specifically when dose-related parameters are estimated, when lag time or infusion duration is estimated, or with large IIV variances (ωk2 > 1) on several estimable parameters; outside these cases OFV comparisons are rarely needed
- For a few individuals with large IIV variances (ωk2 > 1), mapbayr is genuinely outperformed by NONMEM, so a lower OFVi for NONMEM is possible

## Evidence

- [readme] Validation study materials repository: [evidence span withheld — anchor status: external_doc]
