---
name: std-propagation-for-flux-ratios
description: Use when you have triplicate 0 h and 72 h extracellular metabolomics
  intensity/concentration measurements in species and medium samples and need to assign
  uncertainty to per-metabolite boundary fluxes from the MINNO eq 1 estimator (e.g.
  purine/pyrimidine fluxes in Borrelia cultures).
when_to_use_negative:
- Input is intracellular or single-time-point data with no paired 0 h vs 72 h comparison
  — the STD propagation and paired t-test assume triplicate time-course pairs.
- Only a single replicate per metabolite is available — the replicate standard deviations
  required by the STD propagation rule are undefined.
- Metabolites are measured in batch rather than in species-plus-medium co-culture
  design — evaporation correction defined in eq 1 does not apply.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MINNO
  role: Defines the eq 1 boundary flux estimator and the medium-evaporation/batch-effect
    correction whose replicate standard deviations are propagated; MINNO is also the
    visualization environment where the resulting boundary fluxes are mapped onto
    the network.
  repo: github.com/LewisResearchGroup/MINNO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:355bad2f5ef3423ced9e5bf1732e32c87ac6ec6ef6153041f62f889ed7185230
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - MINNO
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# std-propagation-for-flux-ratios

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Propagates replicate standard deviations of 0 h vs 72 h metabolite concentrations through the MINNO metabolic boundary flux estimator (eq 1), yielding a flux uncertainty for each metabolite (e.g. extracellular metabolites measured in specific-species and medium samples over 0–72 h) that is then combined with a paired t-test (α = 0.05) and z-score screen to flag metabolites whose fluxes are statistically significant.

## When to use

Use when you have triplicate 0 h and 72 h extracellular metabolomics intensity/concentration measurements in species and medium samples and need to assign uncertainty to per-metabolite boundary fluxes from the MINNO eq 1 estimator (e.g. purine/pyrimidine fluxes in Borrelia cultures).

## When NOT to use

- Input is intracellular or single-time-point data with no paired 0 h vs 72 h comparison — the STD propagation and paired t-test assume triplicate time-course pairs.
- Only a single replicate per metabolite is available — the replicate standard deviations required by the STD propagation rule are undefined.
- Metabolites are measured in batch rather than in species-plus-medium co-culture design — evaporation correction defined in eq 1 does not apply.

## Inputs

- 0 h triplicate extracellular metabolite concentrations from species samples (e.g. Borrelia culture medium sampled at t=0)
- 72 h triplicate extracellular metabolite concentrations from species samples
- 0 h triplicate metabolite concentrations from cell-free medium controls (for evaporation/batch-effect correction)
- 72 h triplicate metabolite concentrations from cell-free medium controls
- Defined incubation interval ΔT (72 h)
- α = 0.05 significance threshold for paired t-test

## Outputs

- Per-metabolite metabolic boundary fluxes (eq 1)
- Per-metabolite propagated standard deviations of fluxes
- Per-metabolite paired t-test p-values and α = 0.05 significance flags
- Per-metabolite z-scores
- boundary_flux_results.csv (well-formed CSV combining all of the above)

## How to apply

For each metabolite, extract the 0 h and 72 h triplicate concentrations from the species sample and medium sample arms, correct the medium arm for evaporation/batch drift as prescribed by MINNO eq 1, and compute the metabolic boundary flux as ΔC_sp/ΔT with ΔT = 72 h. Propagate the replicate standard deviations through eq 1 using the STD propagation rule (standard error-of-difference form: combine the species and medium replicate SDs for the 0 h and 72 h states into a single flux standard deviation) rather than reporting a point flux alone. Then run a paired t-test (α = 0.05) on the 0 h vs 72 h values to flag metabolites whose concentrations change significantly over the interval, and compute a z-score for each metabolite flux to rank it against the background flux distribution. Write boundary_fluxes, propagated standard deviations, t-test p-values, significance flags, and z-scores to boundary_flux_results.csv; use the t-test and z-score together to distinguish fluxes that are both statistically significant and quantitatively distinct from the bulk.

## Related tools

- **MINNO** (Defines the eq 1 boundary flux estimator and the medium-evaporation/batch-effect correction whose replicate standard deviations are propagated; MINNO is also the visualization environment where the resulting boundary fluxes are mapped onto the network.) — github.com/LewisResearchGroup/MINNO

## Examples

```
python minno/boundary_flux.py --input metabolomics_0h_72h_triplicates.csv --output boundary_flux_results.csv --alpha 0.05
```

## Evaluation signals

- boundary_flux_results.csv exists and is well-formed CSV with columns for flux, propagated SD, t-test p-value, significance flag, and z-score.
- Deterministic synthetic fixture reproduces the module's expected flux values when eq 1 is applied to 0 h vs 72 h triplicate concentrations.
- Propagated standard deviations match the STD propagation rule applied to the replicate SDs (re-derivable from the raw triplicates).
- Significance flags correspond to t-test p-values below α = 0.05 for exactly those metabolites flagged.
- Z-scores computed from the flux distribution are finite and consistent with the propagated SDs for each metabolite.

## Limitations

- Method is defined for extracellular (boundary) fluxes over a single 0 h → 72 h interval; it does not propagate uncertainty through intracellular fluxes or multi-interval time courses.
- Requires triplicate replicates at both time points and both species and medium samples; replicate SDs are undefined for singletons.
- Evaporation/batch-effect normalization is specific to the MINNO eq 1 formulation; applying the STD propagation to fluxes from a different estimator requires re-deriving the propagation rule.
- Statistical support for missing-reaction inference (e.g. purine/pyrimidine reactions in Borrelia) rests on boundary flux patterns alone and should be confirmed against primary literature or annotation databases such as KEGG or NCBI-RefSeq.

## Evidence

- [results] Eq 1 estimates the metabolic boundary flux of metabolite X using the change in its concentration in specific species samples (ΔCsp) and the change in medium samples (ΔCmed) between t = 0 and 72 h, with ΔT = 72 h.: [evidence span withheld — anchor status: not_found]
- [results] Medium-evaporation and batch normalization are part of the boundary flux estimator.: [evidence span withheld — anchor status: not_found]
- [results] Screen metabolites with a paired t-test at alpha = 0.05 comparing 0 h vs 72 h values and compute a z-score for each metabolite's flux.: [evidence span withheld — anchor status: not_found]
