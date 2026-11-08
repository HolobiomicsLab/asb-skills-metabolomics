---
name: ofv-comparison-visualization
description: Use when mapbayr and NONMEM produce discordant individual eta estimates
  (e.g. lag-time/infusion-duration models, dose-related parameters, or large IIV)
  and you need to determine which estimates are more likely by comparing objective
  function values per individual.
when_to_use_negative:
- Estimates are concordant between mapbayr and NONMEM (98% of test-model estimations)
  — there is no OFV adjudication to perform
- You need to assess estimation uncertainty or the ability to return true individual
  eta values — OFV comparison cannot inform on eta accuracy
- Your mapbayr version's compute_ofv machinery does not support the model's log-transformation
  or compartment flags, making the OFVi,NM-like recomputation unreliable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0091
tools:
- name: mapbayr
  role: Performs MAP-Bayesian estimation (mapbayest()) and supplies the compute_ofv
    machinery (omega inverse, sigma, compartment and log-transformation flags) used
    to recompute OFVi from NONMEM etas
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Implements the population PK model and solves the ODEs needed to evaluate
    OFVi for given eta estimates
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Provides the reference posterior eta estimates and original OFVi for the discordant
    individuals
- name: ggplot2
  role: Renders the OFV-comparison scatter figure (ofv_comparison.png)
- name: R
  role: Language in which the comparison pipeline, tallying, and figure rendering
    are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:9efd4a117dd0f1f2f8f060e3e529dd772c005f1ae3857c19cb22f60e9cd3b961
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
  - ggplot2
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ofv-comparison-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recompute and visualize individual objective function values (OFVi) for discordant MAP-Bayesian estimates, comparing mapbayr's OFVi,MBR against an OFVi,NM-like recomputed from NONMEM's posterior eta estimates to adjudicate which tool found the more likely etas.

## When to use

Use when mapbayr and NONMEM produce discordant individual eta estimates (e.g. lag-time/infusion-duration models, dose-related parameters, or large IIV) and you need to determine which estimates are more likely by comparing objective function values per individual.

## When NOT to use

- Estimates are concordant between mapbayr and NONMEM (98% of test-model estimations) — there is no OFV adjudication to perform
- You need to assess estimation uncertainty or the ability to return true individual eta values — OFV comparison cannot inform on eta accuracy
- Your mapbayr version's compute_ofv machinery does not support the model's log-transformation or compartment flags, making the OFVi,NM-like recomputation unreliable

## Inputs

- mapbayr estimation outputs (mapbayests objects / .rds files) for the discordant individuals
- NONMEM estimation outputs (.zip archive of outputs, .mod control streams) containing posterior eta estimates
- mrgsolve model definitions (.cpp files) with $OMEGA, $SIGMA, [ADM]/[OBS] compartment flags and log-transformation settings
- Simulated PK profile datasets (.csv, NM-TRAN format) for Runs 3, 4, and 6 with rich sampling

## Outputs

- ofv_comparison.csv — per-patient table of OFVi,MBR vs recomputed OFVi,NM-like for each discordant individual
- ofv_comparison.png — ggplot2 figure comparing mapbayr vs NONMEM-like OFV
- Fraction of discordant patients with lower OFVi,MBR (e.g. 225/371 ≈ 61%)

## How to apply

Load the validation-study materials from the mapbayr-CPTPSP-2021 repository and select the discordant individuals (e.g. the 371 discordant patients in Runs 3, 4, and 6 with rich sampling). Extract NONMEM's posterior eta estimates along with the corresponding mrgsolve model definition (omega matrix, sigma, residual-error and log-transformation settings). Recompute OFVi,NM-like from the NONMEM etas using mapbayr's compute_ofv machinery (mrgsolve model, omega inverse, sigma, observation/dose compartment flags, log-transformation flag). For each discordant patient, compare the recomputed OFVi,NM-like against OFVi,MBR and tally the fraction with lower OFVi,MBR — in the published analysis 225 of 371 (61%) favored mapbayr. Assemble the per-patient comparison into ofv_comparison.csv and render an OFV-comparison scatter figure with ggplot2 as ofv_comparison.png.

## Related tools

- **mapbayr** (Performs MAP-Bayesian estimation (mapbayest()) and supplies the compute_ofv machinery (omega inverse, sigma, compartment and log-transformation flags) used to recompute OFVi from NONMEM etas) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Implements the population PK model and solves the ODEs needed to evaluate OFVi for given eta estimates) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Provides the reference posterior eta estimates and original OFVi for the discordant individuals)
- **ggplot2** (Renders the OFV-comparison scatter figure (ofv_comparison.png))
- **R** (Language in which the comparison pipeline, tallying, and figure rendering are executed)

## Examples

```
devtools::install_github("FelicienLL/mapbayr"); my_est <- mapbayest(my_model, data = my_data); get_eta(my_est)
```

## Evaluation signals

- The fraction of discordant patients with lower OFVi,MBR reproduces the reported ≈61% (225 of 371) for Runs 3, 4, and 6 with rich sampling
- ofv_comparison.csv contains one row per discordant individual with paired OFVi,MBR and OFVi,NM-like values and no missing etas
- Recomputed OFVi,NM-like values use the exact model settings (omega matrix, sigma, ADM/OBS compartment flags, log-transformation flag) from the corresponding NONMEM control stream
- ofv_comparison.png renders without error and the scatter of mapbayr vs NONMEM-like OFV visually identifies which estimates lie in the lower-OFV region
- Overall concordance statistics remain consistent with the article's reported 98% identical estimations for the test models when discordant cases are excluded

## Limitations

- The proprietary OFVi,NM-like computation could be false, undermining the comparison of estimates
- Discordances were specifically observed when dose-related parameters were estimated, with lag time or infusion duration estimation, or with large IIV variances (ωk2 > 1) — OFV comparison was explored only for these specific cases
- No proper comparison of run times between mapbayr and NONMEM was performed, and mapbayr estimation is acknowledged as slower

## Evidence

- [readme] README description of mapbayr-CPTPSP-2021 repository contents: [evidence span withheld — anchor status: external_doc]
