---
name: pk-profile-simulation-with-interindividual-variability
description: Use when you need to generate simulated TDM-style concentration datasets
  from a published population PK model — with IIV sampled from an omega matrix — to
  validate Bayesian estimation designs or benchmark MAP-BE tools.
when_to_use_negative:
- Your goal is building or fitting a population PK model from raw data (use nlmixr,
  saemix, or NONMEM population estimation instead)
- You already have clinical TDM observations — MAP-BE with mapbayest() on observed
  concentrations is the appropriate skill, not forward simulation
- The model cannot be expressed in mrgsolve code or lacks a specified omega matrix,
  since IIV sampling depends on it
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: mrgsolve
  role: Codes the population PK model and simulates individual concentration profiles
    by solving ODEs with ETAs drawn from the omega matrix
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: mapbayr
  role: Consumes the simulated profiles via mapbayest() for MAP-BE; check_mapbayr_model()
    validates the mrgsolve model code before simulation
  repo: https://github.com/FelicienLL/mapbayr
- name: NONMEM
  role: Receives the same simulated datasets in benchmarking runs with matching LSODA
    solver settings
- name: R
  role: Hosts the simulation workflow (mrgsolve/mcode, data_set, mrgsim) and downstream
    outcome derivation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:e98cca018ee41c223d0521e76d8a806a7e6982bfebabb8428cdc9316db3cacec
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FelicienLL/mapbayr
  repo_url: https://github.com/metrumresearchgroup/mrgsolve
  tools:
  - NONMEM
  - R
  - dplyr
  - ggplot2
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# PK profile simulation with interindividual variability

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate individual PK concentration profiles from a population PK model coded in mrgsolve by sampling ETAs from the published omega matrix and drawing observations according to a therapeutic drug monitoring (TDM) design, as done in the mapbayr validation study (1000 individuals per real model; 4000 profiles per test model).

## When to use

Use when you need to generate simulated TDM-style concentration datasets from a published population PK model — with IIV sampled from an omega matrix — to validate Bayesian estimation designs or benchmark MAP-BE tools.

## When NOT to use

- Your goal is building or fitting a population PK model from raw data (use nlmixr, saemix, or NONMEM population estimation instead)
- You already have clinical TDM observations — MAP-BE with mapbayest() on observed concentrations is the appropriate skill, not forward simulation
- The model cannot be expressed in mrgsolve code or lacks a specified omega matrix, since IIV sampling depends on it

## Inputs

- Published population PK model structure (structural model, omega matrix, sigma/residual error model, TDM design)
- mrgsolve model code ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE blocks)
- Dosing/sampling design specification (single vs multiple dosing, rich vs sparse sampling, infusion rates)
- Validation materials from the mapbayr-CPTPSP-2021 repository (data as .csv, mapbayr code as .cpp, NONMEM code as .mod)

## Outputs

- Simulated PK profiles for ~1000 individuals per model with ETAs drawn from the omega matrix (NM-TRAN-format data.frame with ID, time, evid, cmt, amt, rate, DV, mdv)
- MIPD-relevant PK outcomes per individual (e.g. AUCtau,SS, trough concentrations, time below 0.2 uM for methotrexate)
- Simulated datasets ready for MAP-BE via mapbayest() or NONMEM

## How to apply

Translate each published population PK model into mrgsolve code ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE), checking it with check_mapbayr_model() and coding parent-plus-metabolite models (e.g. sunitinib) with simultaneous compartments. Sample ETAs from the published omega matrix — the mapbayr study simulated 1000 individuals per real published model (4000 profiles per test model combining single/multiple dosing and rich/sparse sampling) — and draw observations according to each drug's published TDM design. Predict concentrations at arbitrary times with mrgsolve's ODE solver (LSODA), and derive MIPD-relevant outcomes such as AUCtau,SS from oral CL and dose, trough concentrations, and time below 0.2 uM for methotrexate. Downstream, the simulated DV values (with mdv=1 for non-observations) are passed to mapbayest() and, for benchmarking, to a matching NONMEM control stream. Correct application means the simulated profiles reproduce the published IIV (omega^2 magnitudes, e.g. >1 for problematic models) and dosing/sampling structure, so that estimations computed on them are comparable across tools.

## Related tools

- **mrgsolve** (Codes the population PK model and simulates individual concentration profiles by solving ODEs with ETAs drawn from the omega matrix) — https://github.com/metrumresearchgroup/mrgsolve
- **mapbayr** (Consumes the simulated profiles via mapbayest() for MAP-BE; check_mapbayr_model() validates the mrgsolve model code before simulation) — https://github.com/FelicienLL/mapbayr
- **NONMEM** (Receives the same simulated datasets in benchmarking runs with matching LSODA solver settings)
- **R** (Hosts the simulation workflow (mrgsolve/mcode, data_set, mrgsim) and downstream outcome derivation)

## Examples

```
my_model <- mcode("Example_model", code); simdat <- updated_model %>% data_set(subset(scenarios, scenario == i)) %>% mrgsim(output = "df", recover = "scenario")
```

## Evaluation signals

- Simulated observation records follow the published TDM design exactly (rich/sparse, single/multiple dosing, correct mdv flags for non-observed rows)
- Distribution of simulated ETAs matches the published omega matrix (variance magnitudes, including omega^2 > 1 cases known to be difficult)
- Model code passes check_mapbayr_model() before simulation; sunitinib uses simultaneous parent and metabolite compartments
- Derived PK outcomes are computable per individual (e.g. AUCtau,SS from oral CL and dose; troughs; time below 0.2 uM for methotrexate), matching the mapbayr-CPTPSP-2021 .csv data structure
- Simulated IPRED/DV values reproduce the model with known ETAs, so subsequent estimation error (e.g. only 5/1000 individuals with >10% AUC error for ibrutinib) is attributable to estimation, not simulation bugs

## Limitations

- Simulation-then-estimation on simulated data cannot assess the ability of MAP-BE to return the true individual parameter value (ηi)
- The Bayesian estimation design must be previously validated, including residual error and sampling strategy validation, before results are trusted
- Models with large IIV variances (omega^2 > 1) on several estimable parameters produced genuine estimation discrepancies, so profiles simulated from such models need cautious interpretation

## Evidence

- [readme] mrgsolve as hierarchical ODE simulation engine: [evidence span withheld — anchor status: external_doc]
- [other] Simulation datasets stored for both tools: [evidence span withheld — anchor status: external_doc]
