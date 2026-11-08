---
name: enrichment-rmse-method-benchmarking
description: Use when simulated or standard-spiked isotope-labeling data with known
  ground-truth enrichment are available and you must decide which correction method
  (modified MDT, modified ULS, original ULS, NRE, FAM) is most accurate for a given
  tracer/resolution combination.
when_to_use_negative:
- The correction method under test cannot handle the tracer element — e.g. the original
  MDT does not support 34S, so it must be excluded from a 34S benchmark
- Ground-truth enrichment is unknown (real biological samples without standards),
  since RMSE against theoretical enrichment cannot be computed
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Runs the modified MDT, original ULS, and modified ULS corrections on simulated
    34S FAM data at resolution 140,000 and computes enrichment
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Provides the NRE combinatorial correction baseline that ignores resolution
    effect
- name: MDT
  role: Mass-difference-theory-based correction implemented in ElemCor; original variant
    excluded for 34S tracers
  repo: https://github.com/4dsoftware/elemcor
- name: ULS
  role: Unlabeled-sample-based correction implemented in ElemCor via deconvolution
    of unlabeled-sample FAM
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Host environment for ElemCor (developed under MATLAB 2016b); MATLAB Compiler
    Runtime is installed to run the executable
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:c41dad49eb1d9bb2f870a2038d98eee43da0627206df9ca786a6cb2f28fa0105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# enrichment-rmse-method-benchmarking

## Summary

Benchmark isotope-labeling correction methods (MDT, ULS, NRE, uncorrected FAM) by computing enrichment RMSE against known theoretical enrichments, e.g. for simulated 34S metabolites at Orbitrap resolution 140,000, to verify a correction implementation reproduces the ElemCor method ranking.

## When to use

Use when simulated or standard-spiked isotope-labeling data with known ground-truth enrichment are available and you must decide which correction method (modified MDT, modified ULS, original ULS, NRE, FAM) is most accurate for a given tracer/resolution combination.

## When NOT to use

- The correction method under test cannot handle the tracer element — e.g. the original MDT does not support 34S, so it must be excluded from a 34S benchmark
- Ground-truth enrichment is unknown (real biological samples without standards), since RMSE against theoretical enrichment cannot be computed

## Inputs

- Simulated FAM column vectors (one per compound per sample) for ten sulfur-containing metabolites with known theoretical enrichments, in XLSX format (test_labelS_sim.xlsx)
- Unlabeled-sample FAM XLSX for deconvolution-based ULS correction (test_unlabelS_sim.xlsx)
- Theoretical enrichment values per metabolite as ground truth

## Outputs

- Per-method enrichment RMSE table for each metabolite (s34_correction_rmse.csv)
- Corrected MDVs and isotopic enrichment values stored as an additional sheet in the input XLSX

## How to apply

Generate FAM vectors (one column per compound per sample) with known theoretical enrichments, mimicking XCalibur/Orbitrap simulations; for 34S at nominal resolution 140,000 load test_labelS_sim.xlsx and test_unlabelS_sim.xlsx. Run ElemCor with 34S selected as tracer, Orbitrap as analyzer, and resolution 140,000 to obtain the modified MDT, original ULS, and modified ULS corrections (skip the original MDT, which does not support 34S tracers). Apply the no-resolution-effect (NRE) combinatorial correction used by IsoCor and keep the uncorrected FAM as a baseline. For each method compute MDVs by non-negative linear regression of the correction-matrix equation followed by normalization to a sum of 1, derive isotopic enrichment per compound, then compute enrichment RMSE against theoretical enrichment per metabolite and method. Accept the benchmark when the modified MDT and modified ULS RMSE are lowest and the original ULS RMSE exceeds the NRE RMSE — the latter because sulfur's most abundant isotope (95.0% 32S) has a lower fractional abundance than nitrogen's (99.6% 14N), so the unlabeled-sample deconvolution for 34S is less reliable.

## Related tools

- **ElemCor** (Runs the modified MDT, original ULS, and modified ULS corrections on simulated 34S FAM data at resolution 140,000 and computes enrichment) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Provides the NRE combinatorial correction baseline that ignores resolution effect)
- **MDT** (Mass-difference-theory-based correction implemented in ElemCor; original variant excluded for 34S tracers) — https://github.com/4dsoftware/elemcor
- **ULS** (Unlabeled-sample-based correction implemented in ElemCor via deconvolution of unlabeled-sample FAM) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Host environment for ElemCor (developed under MATLAB 2016b); MATLAB Compiler Runtime is installed to run the executable)

## Examples

```
ElemCor.exe test_labelS_sim.xlsx test_unlabelS_sim.xlsx --tracer 34S --analyzer Orbitrap --resolution 140000 --purity 0.95
```

## Evaluation signals

- Modified MDT and modified ULS yield lower enrichment RMSE than NRE and uncorrected FAM
- Original ULS RMSE exceeds the NRE RMSE (mirrors the paper's reported ranking for 34S)
- Each MDV after non-negative linear regression sums to 1 after normalization
- FAM vector length equals Nt + 1 (M+0 … M+Nt) for the ten sulfur metabolites
- Results are written as a per-method RMSE comparison table (e.g. s34_correction_rmse.csv)

## Limitations

- ULS requires the additional experimental measurement of unlabeled samples, adding cost; MDT is recommended for low-mass compounds instead
- If the wrong tracer element is selected (e.g. C or S selected while 15N data are loaded), ElemCor errors out because some compounds lack the tracer element in their formula
- ElemCor is developed in MATLAB 2016b and distributed as a compiled executable requiring MATLAB Compiler Runtime installation

## Evidence

- [abstract] MDT and ULS methods were validated with simulated data and experimental data from Saccharomyces cerevisiae, showing improvement in accuracy and comprehensiveness over existing methods: "We validate both methods and show their improvement in accuracy and comprehensiveness over existing methods using simulated data and experimental data from Saccharomyces cerevisiae."
- [discussion] Non-negative linear regression obtains MDV and enrichment: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] Steps 1–6 of the ElemCor GUI workflow including tracer and resolution specification: "In Steps 1 and 2, labeled and unlabeled data are loaded. Step 2 is optional, and when it is not performed, ElemCor runs based on mass difference theory only. In Step 3, isotopic purity of nutrient"
- [readme] ElemCor supports 18O and 34S as tracer elements beyond 13C, 2H, 15N: "In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction. PLEASE MAKE SURE THE CORRECT TRACER IS SELECTED!"
- [readme] Example XLSX files simulate N and S tracers at resolutions 140,000 and 280,000 on an Orbitrap: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively. Both are from Orbitrap analyzer."
- [discussion] IsoCor performs correction without considering resolution effect (NRE): "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [readme] FAM length can differ from Nt + 1 and ElemCor handles it: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms. But ElemCor is able to handle FAM that has a length other than Nt + 1"
