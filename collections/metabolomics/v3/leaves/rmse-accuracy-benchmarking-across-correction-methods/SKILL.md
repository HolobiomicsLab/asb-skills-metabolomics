---
name: rmse-accuracy-benchmarking-across-correction-methods
description: Use when you have simulated or spike-in labeled metabolite data with
  known theoretical enrichment and need to quantify which natural-abundance/resolution
  correction method (MDT, ULS, NRE/IsoCor, uncorrected FAM) recovers enrichment most
  accurately.
when_to_use_negative:
- Data are low-resolution or at extremely low instrument resolution, where ElemCor
  is identical to IsoCor/NRE and the resolution-effect comparison has no discriminating
  power
- The goal is correcting a real experimental dataset rather than benchmarking correction
  accuracy — use the corrected MDVs directly instead of building an RMSE comparison
- Tracer element is absent from some compound formulas (e.g., selecting C or S as
  tracer on 15N data), which the algorithm cannot handle
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: ElemCor
  role: Provides the modified MDT and ULS resolution-dependent corrections being benchmarked,
    plus corrected MDVs and enrichment calculation
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Environment in which ElemCor (2016b) and its MATLAB functions run; correction
    matrices and non-negative regression are computed in MATLAB
- name: IsoCor
  role: Reference implementation of the NRE combinatorial (resolution-ignorant) correction
    used as the accuracy baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:426ac05e4654361d40548a10365ca686c095f73ff31aa3250517b457d5eac7ee
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# rmse-accuracy-benchmarking-across-correction-methods

## Summary

A benchmarking protocol comparing isotope-labeling correction methods (ElemCor's modified MDT and ULS vs. raw FAM, NRE/IsoCor-style combinatorial correction, and the original MDT/ULS implementations) by computing per-metabolite RMSE of recovered isotopic enrichment against a known theoretical enrichment (e.g., 20% 15N) on simulated high-resolution LC-MS data.

## When to use

Use when you have simulated or spike-in labeled metabolite data with known theoretical enrichment and need to quantify which natural-abundance/resolution correction method (MDT, ULS, NRE/IsoCor, uncorrected FAM) recovers enrichment most accurately.

## When NOT to use

- Data are low-resolution or at extremely low instrument resolution, where ElemCor is identical to IsoCor/NRE and the resolution-effect comparison has no discriminating power
- The goal is correcting a real experimental dataset rather than benchmarking correction accuracy — use the corrected MDVs directly instead of building an RMSE comparison
- Tracer element is absent from some compound formulas (e.g., selecting C or S as tracer on 15N data), which the algorithm cannot handle

## Inputs

- Theoretical FAM vectors (length Nt+1, M+0..M+Nt) for labeled metabolites at known theoretical enrichment (e.g., test_labelN_sim.xlsx format, 15N tracer, 20% enrichment)
- Unlabeled-sample FAM file for ULS deconvolution (e.g., test_unlabelN_sim.xlsx format)
- Tracer element, nominal instrument resolution (e.g., 140,000 Orbitrap), mass analyzer, and tracer isotopic purity specifications

## Outputs

- Per-metabolite, per-method RMSE table of recovered isotopic enrichment vs. theoretical enrichment (e.g., rmse_15n_correction_methods.csv)
- Corrected MDVs and enrichment values per compound/method for each correction approach

## How to apply

Generate theoretical FAM vectors of length Nt+1 (covering M+0 through M+Nt) for each tracer-element-labeled metabolite at the chosen theoretical enrichment (e.g., 24 15N metabolites at 20% enrichment, nominal Orbitrap resolution 140,000), using the packaged test_labelN_sim.xlsx / test_unlabelN_sim.xlsx format where each column is a compound's FAM in a sample. Apply each correction method: (a) uncorrected FAM, (b) NRE combinatorial correction as in IsoCor that ignores resolution effects, (c) original mass-difference-theory (MDT) correction via non-negative linear regression of the correction matrix followed by normalization to a sum of 1, (d) original ULS correction by deconvolving the unlabeled-sample FAM Nt times, and (e) ElemCor's modified MDT and modified ULS corrections that account for the resolution effect at the specified nominal resolution. After each correction, compute isotopic enrichment per metabolite and the RMSE of enrichment versus the theoretical value for every metabolite-method pair. Judgment: the modified MDT and modified ULS should yield significantly lower RMSE than FAM, NRE, and the original MDT/ULS implementations; write the per-metabolite, per-method RMSE table (e.g., rmse_15n_correction_methods.csv) as validation output.

## Related tools

- **ElemCor** (Provides the modified MDT and ULS resolution-dependent corrections being benchmarked, plus corrected MDVs and enrichment calculation) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Environment in which ElemCor (2016b) and its MATLAB functions run; correction matrices and non-negative regression are computed in MATLAB)
- **IsoCor** (Reference implementation of the NRE combinatorial (resolution-ignorant) correction used as the accuracy baseline)

## Evaluation signals

- Modified MDT and modified ULS corrections yield significantly lower RMSE from the 20% theoretical enrichment than raw FAM and NRE for all 24 simulated 15N metabolites
- Modified methods also outperform the original (mathematically defective) MDT and ULS implementations
- Corrected MDVs sum to 1 after non-negative linear regression followed by normalization
- Processed example files (test_labelN_sim.xlsx at 140,000 Orbitrap resolution) complete within ~10 seconds on a 2.6GHz Core i7, with results written to additional sheets of the same XLSX file
- FAM vector lengths equal Nt+1 (M+0 through M+Nt) for each metabolite before correction

## Limitations

- Existing implementations of both the unlabeled-sample-based and mass-difference-theory correction methods have mathematical defects; the original MDT/ULS should not be treated as ground truth
- ULS requires the additional experimental cost of measuring unlabeled samples; MDT is recommended for small metabolites and ULS for high-mass compounds with large spectral inaccuracy
- No public accession or availability statement covers the simulated/experimental validation datasets beyond the packaged example XLSX files; ElemCor depends on a proprietary MATLAB (2016b) environment
- The amount of correction is monotonically dependent on instrument resolution: at extremely high resolution ElemCor performs no correction at all, limiting benchmark sensitivity

## Evidence

- [results] For all 24 simulated 15N metabolites at all degrees of theoretical enrichment, ElemCor's MDT and ULS corrections yielded significantly lower RMSE: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [methods] RMSE benchmarking protocol: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [readme] FAM vector format: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [readme] Resolution dependence of correction: "Therefore, the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [methods] Dataset format: "The package includes example data files from labeled samples "test_labelN_sim.xlsx" and "test_labelS_sim.xlsx""
