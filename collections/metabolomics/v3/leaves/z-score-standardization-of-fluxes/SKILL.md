---
name: z-score-standardization-of-fluxes
description: Use when you have triplicate metabolite intensity measurements at 0 h
  and 72 h (culture and medium samples) and need to convert raw boundary flux estimates
  into standardized z-scores for cross-metabolite or cross-species comparison, e.
when_to_use_negative:
- When the data are absolute intracellular concentrations rather than 0 h vs 72 h
  extracellular (medium-corrected) changes — the flux estimator and its STD propagation
  assume the ΔCsp vs ΔCmed formulation of eq 1.
- When you need pathway-level reaction predictions rather than standardized flux magnitudes
  — use MINNO's boundary-flux-to-missing-reaction inference instead of z-score standardization.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: MINNO
  role: Web-based mapping tool that computes metabolite boundary fluxes from the temporal
    metabolite intensity profiles and supports their standardization for comparison
    across species; used to evaluate the 0 h vs 72 h flux estimator and t-test/z-score
    outputs
  repo: https://github.com/LewisResearchGroup/MINNO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:35146820a439493e53c4c607af4a75809421b9341f4d6c0eb8597c1a5edb44cf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  repo_url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - MINNO
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# z-score-standardization-of-fluxes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes z-scores for metabolite boundary fluxes estimated from 0 h vs 72 h extracellular metabolomics data, using the medium-evaporation-corrected flux estimator (eq 1) with STD propagation of triplicate standard deviations, so that flux magnitudes can be compared across metabolites and species and flagged reactions validated.

## When to use

Use when you have triplicate metabolite intensity measurements at 0 h and 72 h (culture and medium samples) and need to convert raw boundary flux estimates into standardized z-scores for cross-metabolite or cross-species comparison, e.g. to rank flux changes or identify metabolites whose fluxes are most anomalous relative to the distribution.

## When NOT to use

- When the data are absolute intracellular concentrations rather than 0 h vs 72 h extracellular (medium-corrected) changes — the flux estimator and its STD propagation assume the ΔCsp vs ΔCmed formulation of eq 1.
- When you need pathway-level reaction predictions rather than standardized flux magnitudes — use MINNO's boundary-flux-to-missing-reaction inference instead of z-score standardization.

## Inputs

- Triplicate extracellular metabolite intensity measurements at 0 h and 72 h for species (sp) and medium (med) samples
- Metabolite-to-reaction mapping or target metabolite list for which boundary fluxes and z-scores are to be computed

## Outputs

- boundary_flux_results.csv with columns for boundary flux, propagated standard deviation, paired t-test p-value (α = 0.05), significance flag, and z-score per metabolite

## How to apply

First load a triplicate 0 h / 72 h metabolomics fixture for each metabolite measured in both species (sp) and medium (med) samples. Apply the medium-evaporation correction and instrumental batch normalization prescribed by eq 1 to the raw intensities, then compute the boundary flux of metabolite X as the difference between ΔCsp and ΔCmed over ΔT = 72 h. Propagate the replicate standard deviations through the flux estimator using the STD propagation rule so each flux carries an associated uncertainty. Screen each metabolite with a paired t-test at α = 0.05 comparing 0 h vs 72 h values, then compute the z-score for each metabolite's flux (flux standardized against the distribution of fluxes across metabolites) to rank fluxes by magnitude of deviation. Write boundary fluxes, propagated standard deviations, t-test p-values, significance flags, and z-scores to boundary_flux_results.csv; the z-score column is the deliverable that enables downstream comparison of, e.g., purine vs pyrimidine flux behavior across Borrelia species.

## Related tools

- **MINNO** (Web-based mapping tool that computes metabolite boundary fluxes from the temporal metabolite intensity profiles and supports their standardization for comparison across species; used to evaluate the 0 h vs 72 h flux estimator and t-test/z-score outputs) — https://github.com/LewisResearchGroup/MINNO

## Examples

```
python scripts/boundary_flux.py --input metabolomics_triplicates.csv --t0 0 --t72 72 --alpha 0.05 --output boundary_flux_results.csv
```

## Evaluation signals

- boundary_flux_results.csv exists and is well-formed CSV with the expected columns (flux, STD, p-value, significance flag, z-score).
- The deterministic triplicate fixture yields the module's expected z-score, paired t-test (α = 0.05), and boundary flux outputs — a unit-level expert review checks the statistical correctness of the flux and z-score computations.
- Flux values match the eq 1 formulation with ΔT = 72 h; any deviation indicates the ΔCsp vs ΔCmed correction or STD propagation was applied incorrectly.
- Every metabolite flagged as significant (α = 0.05) has a correspondingly large |z-score|, confirming internal consistency between the t-test and z-score columns.
- The z-score distribution across metabolites should be approximately centered near zero with unit variance for the fixture, verifying that standardization was performed correctly.

## Limitations

- The z-score normalization is only meaningful when fluxes come from the same medium-evaporation and batch-corrected estimator; mixing fluxes from different estimators would break comparability.
- Supporting validation results (e.g. expected flux/z-score tables) reside in supplementary Table S1, which is not included in the article body, so the expected z-score range must be validated against the deterministic fixture rather than published values.
- No version number for the MINNO software is stated, so reproducibility of the flux/z-score pipeline depends on pinning the specific commit or release used.

## Evidence

- [results] Eq 1 estimates the metabolic boundary flux of metabolite X using the change in its concentration in specific species samples (ΔCsp) and the change in medium samples (ΔCmed) between t = 0 and 72 h, with ΔT = 72 h.: [evidence span withheld — anchor status: not_found]
- [results] paired t-test at alpha = 0.05 comparing 0 h vs 72 h values and compute a z-score for each metabolite's flux: [evidence span withheld — anchor status: not_found]
