---
name: paired-t-test-metabolite-screening
description: Use when comparing replicate metabolite intensities measured on the same
  cultures at two time points (e.g. 0 h vs 72 h extracellular samples) and you must
  flag which metabolites show significant change before computing boundary fluxes.
when_to_use_negative:
- Data are unpaired or the two time points come from different culture batches — a
  paired t-test is then inappropriate.
- The goal is reaction-gap annotation rather than significance screening; skip straight
  to the boundary-flux inference step.
- Only a single time-point measurement exists (no paired 0 h reference) — use a one-sample
  or single-factor test instead.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: MINNO
  role: Hosts the temporal metabolite-intensity profiles and boundary-flux computation
    that the paired t-test significance screening feeds into.
  repo: github.com/LewisResearchGroup/MINNO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:e6da56a44f0c9af362acd787d4508a3d6456026ae8f02a91515ba16897b6ac2d
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# paired-t-test-metabolite-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Screen temporal metabolomics measurements for statistically significant change by applying a paired t-test at α = 0.05 to 0 h vs 72 h intensities, then attaching z-scores to each metabolite's boundary flux. This is the significance-testing layer of the MINNO boundary-flux workflow that separates real consumption/production from noise.

## When to use

Use when comparing replicate metabolite intensities measured on the same cultures at two time points (e.g. 0 h vs 72 h extracellular samples) and you must flag which metabolites show significant change before computing boundary fluxes.

## When NOT to use

- Data are unpaired or the two time points come from different culture batches — a paired t-test is then inappropriate.
- The goal is reaction-gap annotation rather than significance screening; skip straight to the boundary-flux inference step.
- Only a single time-point measurement exists (no paired 0 h reference) — use a one-sample or single-factor test instead.

## Inputs

- Deterministic synthetic or real metabolomics fixture with 0 h and 72 h triplicate intensity measurements per metabolite
- Raw triplicate extracellular metabolite intensities requiring medium-evaporation correction and batch normalization

## Outputs

- boundary_flux_results.csv containing per-metabolite boundary fluxes, propagated standard deviations, paired t-test p-values, significance flags, and z-scores

## How to apply

1. Generate or load a deterministic synthetic metabolomics fixture with 0 h and 72 h triplicate intensity measurements per metabolite, modeled on the Methods experimental design. 2. Pair the 0 h and 72 h measurements per metabolite and apply the medium-evaporation correction and batch normalization to raw triplicate intensities as prescribed by eq 1 of the MINNO workflow. 3. Run a paired t-test at alpha = 0.05 comparing 0 h vs 72 h values for each metabolite and record the p-value. 4. Compute the boundary flux for each metabolite via eq 1, propagate replicate standard deviations with the STD propagation rule, and compute a z-score for each metabolite's flux. 5. Write boundary fluxes, propagated SDs, t-test p-values, significance flags, and z-scores to boundary_flux_results.csv. Rationale: paired design accounts for same-culture baseline variability, and the significance flag prevents missing-reaction inference from riding on non-significant intensity drift.

## Related tools

- **MINNO** (Hosts the temporal metabolite-intensity profiles and boundary-flux computation that the paired t-test significance screening feeds into.) — github.com/LewisResearchGroup/MINNO

## Examples

```
python minno_boundary_flux.py --input metabolomics_fixture.csv --alpha 0.05 --output boundary_flux_results.csv
```

## Evaluation signals

- boundary_flux_results.csv exists and is well-formed CSV
- The deterministic fixture reproduces the module's expected flux, t-test (α = 0.05), and z-score outputs
- Significance flags correspond to p < 0.05 in the paired t-test column
- Propagated standard deviations follow the STD propagation rule for replicate variability
- Z-scores are finite and consistent with the flux and propagated SD for each metabolite

## Limitations

- The article does not state the exact multiple-testing correction used, so α = 0.05 per-metabolite is the assumed threshold.
- Supporting statistics for the Borrelia reaction-gap findings reside in supplementary Table S1, which is not included in the section text.
- The workflow assumes replicate structure (triplicates) and paired time-point sampling; deviations from this design will invalidate the paired t-test.

## Evidence

- [methods] Paired t-test screen: [evidence span withheld — anchor status: not_found]
- [methods] Boundary flux definition: [evidence span withheld — anchor status: not_found]
