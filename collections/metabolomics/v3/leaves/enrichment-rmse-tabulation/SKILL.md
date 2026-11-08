---
name: enrichment-rmse-tabulation
description: Use when comparing the accuracy of natural-abundance/resolution-effect
  correction methods on simulated or experimental isotope-labeling data, where corrected
  enrichment can be scored against a known theoretical enrichment (e.g. 20% 13C/15N/34S).
when_to_use_negative:
- Data are already corrected MDVs or enrichments — RMSE against theoretical values
  would conflate upstream errors; only raw/uncorrected FAM should feed this benchmark
- No theoretical or known true enrichment exists for the samples (e.g. unlabeled or
  fully labeled extremes absent) — RMSE against truth cannot be computed and this
  skill is not applicable
- The tracer element is not present in the compound's formula (ElemCor reports an
  error, e.g. selecting C or S as tracer for 15N data)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Applies MDT- and ULS-based resolution-dependent correction and computes isotopic
    enrichment per compound from loaded XLSX FAM data
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass-difference-theory correction arm of ElemCor used as one benchmark method
    (no unlabeled sample required)
  repo: https://github.com/4dsoftware/elemcor
- name: ULS
  role: Unlabeled-sample-based correction arm of ElemCor (improved original ULS) used
    as a benchmark method requiring test_unlabel*_sim.xlsx inputs
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Host environment for ElemCor (developed under MATLAB 2016b); MATLAB functions
    can be scripted to batch-run corrections before RMSE computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:c13a66bba2a1b6ec4ee4144d09a49240b4ed6c629418fbcc33ddfe6c553e478a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# enrichment-rmse-tabulation

## Summary

Benchmark isotope-correction methods (ElemCor MDT, ElemCor ULS, original ULS, NRE/IsoCor-style, and no correction) by computing RMSE of corrected fractional-abundance enrichment against theoretical enrichment, then tabulating compound-tracer-method RMSE values (e.g. for CoA metabolites under 13C/15N/34S tracers at resolution 280,000).

## When to use

Use when comparing the accuracy of natural-abundance/resolution-effect correction methods on simulated or experimental isotope-labeling data, where corrected enrichment can be scored against a known theoretical enrichment (e.g. 20% 13C/15N/34S).

## When NOT to use

- Data are already corrected MDVs or enrichments — RMSE against theoretical values would conflate upstream errors; only raw/uncorrected FAM should feed this benchmark
- No theoretical or known true enrichment exists for the samples (e.g. unlabeled or fully labeled extremes absent) — RMSE against truth cannot be computed and this skill is not applicable
- The tracer element is not present in the compound's formula (ElemCor reports an error, e.g. selecting C or S as tracer for 15N data)

## Inputs

- Simulated or experimental FAM matrix (XLSX, one column per sample, rows = M+0..M+Nt isotopologs) for each compound and tracer
- Optional unlabeled-sample FAM files for ULS-based correction (e.g. test_unlabelN_sim.xlsx / test_unlabelS_sim.xlsx)
- Theoretical enrichment level used in simulation (e.g. 20%)
- Compound formulas and tracer element identity (13C, 15N, 34S) and nominal instrument resolution (e.g. 140,000 or 280,000)

## Outputs

- rmse_table.csv with RMSE of corrected enrichment vs. theoretical enrichment for each compound-tracer-method combination
- Corrected MDVs and enrichment values per compound and sample (stored as new sheets in the input XLSX by ElemCor)

## How to apply

First generate or collect simulated fractional abundances of measured isotopologs (FAM) for each compound-tracer combination — e.g. coenzyme A, acetyl-CoA, succinyl-CoA, and HMG-CoA under 13C, 15N, and 34S tracers at nominal instrument resolution 280,000 on an Orbitrap analyzer. Run each correction method through ElemCor (load labeled and optional unlabeled data, specify isotopic purity and nominal resolution, select the correct tracer element and mass analyzer, then analyze), plus the original ULS scheme, a no-resolution-effect (NRE, IsoCor-style) correction, and no correction. For each corrected mass distribution vector, compute isotopic enrichment and take the RMSE against the theoretical enrichment used in the simulation (20% in the published validation). Tabulate one RMSE per compound-tracer-method combination and save as a CSV (rmse_table.csv) so that per-tracer patterns — such as original ULS being excellent for 15N but degraded for 13C and 34S — can be read directly.

## Related tools

- **ElemCor** (Applies MDT- and ULS-based resolution-dependent correction and computes isotopic enrichment per compound from loaded XLSX FAM data) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass-difference-theory correction arm of ElemCor used as one benchmark method (no unlabeled sample required)) — https://github.com/4dsoftware/elemcor
- **ULS** (Unlabeled-sample-based correction arm of ElemCor (improved original ULS) used as a benchmark method requiring test_unlabel*_sim.xlsx inputs) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Host environment for ElemCor (developed under MATLAB 2016b); MATLAB functions can be scripted to batch-run corrections before RMSE computation)

## Examples

```
ElemCor.exe  # load test_labelS_sim.xlsx + test_unlabelS_sim.xlsx, purity 0.99, resolution 280000, tracer=S, analyzer=Orbitrap; then compute RMSE vs 20% and write rmse_table.csv
```

## Evaluation signals

- RMSE for ElemCor MDT/ULS on 13C and 34S CoA-metabolite simulations at resolution 280,000 should be markedly lower than the original ULS method's RMSE for those tracers
- Original ULS RMSE for 15N should remain low (comparable to ElemCor), reproducing the published pattern
- NRE-style correction (IsoCor combinatorics, no resolution effect) should show higher RMSE than resolution-aware methods at nominal resolution 280,000
- Corrected MDVs must sum to 1 after non-negative linear regression followed by normalization (ElemCor's documented normalization step)
- ElemCor should complete the example 15N/34S test files within ~10 s on a 2.6 GHz Core i7; results are written to new sheets in the original XLSX file

## Limitations

- ULS requires additional unlabeled-sample measurements, adding experimental cost; MDT is recommended for low-mass compounds for cost efficiency
- ElemCor depends on a proprietary MATLAB (2016b) / MATLAB Compiler Runtime environment, which may limit reproducibility of the benchmark
- The tracer element selected must match the data — loading 15N data while selecting C or S causes the software to fail or run improperly when the tracer element is absent from the compound formula
- Experimental validation in the source article covered only S. cerevisiae with 15N ammonium; 13C/34S benchmark behavior rests on simulations

## Evidence

- [results] In the simulations of the four CoA metabolites, the original ULS method was excellent for 15N but remarkably lower for 13C and 34S, while those tracers were accurately calculated by ElemCor.: [evidence span withheld — anchor status: not_found]
- [discussion] correction without considering resolution effect: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [readme] They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively.: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
- [readme] In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified.: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
