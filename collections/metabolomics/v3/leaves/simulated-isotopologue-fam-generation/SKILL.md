---
name: simulated-isotopologue-fam-generation
description: Use when benchmarking isotopic-enrichment correction methods (MDT, ULS,
  NRE/IsoCor, FAM-only) against a known ground-truth enrichment, requiring simulated
  labeled and unlabeled FAM inputs at a specified nominal resolution (e.g., 140,000).
when_to_use_negative:
- When correcting real experimental LC-MS isotope-labeling data rather than benchmarking
  correction methods against known ground-truth enrichment — use ElemCor's correction
  workflow directly instead
- When the tracer element is not present in a compound's formula (e.g., selecting
  C or S as tracer for 15N-only compounds) — the algorithm will report an error
- When the instrument operates at extremely low resolution, where ElemCor's resolution-dependent
  correction is identical to IsoCor's NRE and the benchmark cannot discriminate methods
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ElemCor
  role: MATLAB package providing modified MDT and ULS resolution-dependent corrections
    applied to the simulated FAMs and used to compute isotopic enrichment; example
    files test_labelN_sim.xlsx/test_unlabelN_sim.xlsx define the simulation format
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory correction via non-negative linear regression of the
    correction matrix followed by normalization to sum 1; one of the benchmarked methods
  repo: https://github.com/4dsoftware/elemcor
- name: ULS
  role: Unlabeled-sample-based correction by deconvolution of the unlabeled-sample
    FAM for the tracer element Nt times; one of the benchmarked methods
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference implementation of the NRE combinatorial correction (ignoring resolution
    effects) used as a baseline in the RMSE comparison
- name: MATLAB
  role: Development environment for ElemCor; MATLAB functions can be adapted into
    the simulation/benchmark pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:da6cf60ef2458f57ff781f8f5d7a8eeb48c8ed78a5abebf704a2511a1af11365
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# simulated-isotopologue-fam-generation

## Summary

Generate theoretical fractional abundances of measured isotopologs (FAM) for simulated stable-isotope-labeled metabolites (e.g., 24 15N metabolites at 20% enrichment, Orbitrap resolution 140,000) to benchmark isotope-correction methods such as ElemCor MDT/ULS against FAM-only, NRE/IsoCor, and original MDT/ULS corrections via RMSE of enrichment.

## When to use

Use when benchmarking isotopic-enrichment correction methods (MDT, ULS, NRE/IsoCor, FAM-only) against a known ground-truth enrichment, requiring simulated labeled and unlabeled FAM inputs at a specified nominal resolution (e.g., 140,000).

## When NOT to use

- When correcting real experimental LC-MS isotope-labeling data rather than benchmarking correction methods against known ground-truth enrichment — use ElemCor's correction workflow directly instead
- When the tracer element is not present in a compound's formula (e.g., selecting C or S as tracer for 15N-only compounds) — the algorithm will report an error
- When the instrument operates at extremely low resolution, where ElemCor's resolution-dependent correction is identical to IsoCor's NRE and the benchmark cannot discriminate methods

## Inputs

- List of tracer-element-labeled metabolite formulas (e.g., 24 15N-enriched metabolites) with tracer atom counts Nt
- Theoretical tracer enrichment level (e.g., 20%) and nominal instrument resolution (e.g., 140,000 Orbitrap)
- ElemCor example format: XLSX files with each column vector the FAM of all compounds in a sample (test_labelN_sim.xlsx / test_unlabelN_sim.xlsx style)

## Outputs

- Simulated labeled-sample FAM file (XLSX, e.g., test_labelN_sim.xlsx format)
- Simulated unlabeled-sample FAM file (XLSX, e.g., test_unlabelN_sim.xlsx format)
- Per-metabolite, per-method RMSE table (e.g., rmse_15n_correction_methods.csv)

## How to apply

Generate theoretical FAM vectors of length Nt+1 (covering M+0 through M+Nt) for each metabolite, using the tracer element's atom count and a theoretical enrichment level (e.g., 20% 15N), simulated at a nominal Orbitrap resolution such as 140,000 in the XCalibur format mirrored by the packaged test_labelN_sim.xlsx / test_unlabelN_sim.xlsx files. Produce paired labeled and unlabeled simulated files so that ULS-based corrections (deconvolution of unlabeled-sample FAM Nt times) can be exercised alongside MDT corrections (non-negative linear regression of the correction matrix followed by normalization to a sum of 1). Apply each correction method — raw FAM, NRE (IsoCor-style combinatorial correction ignoring resolution), original and modified MDT, original and modified ULS — then compute isotopic enrichment per metabolite and the RMSE against the theoretical 20% enrichment for every metabolite-method pair. Validation succeeds when modified MDT and modified ULS show significantly lower RMSE than FAM, NRE, and the original implementations.

## Related tools

- **ElemCor** (MATLAB package providing modified MDT and ULS resolution-dependent corrections applied to the simulated FAMs and used to compute isotopic enrichment; example files test_labelN_sim.xlsx/test_unlabelN_sim.xlsx define the simulation format) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory correction via non-negative linear regression of the correction matrix followed by normalization to sum 1; one of the benchmarked methods) — https://github.com/4dsoftware/elemcor
- **ULS** (Unlabeled-sample-based correction by deconvolution of the unlabeled-sample FAM for the tracer element Nt times; one of the benchmarked methods) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Reference implementation of the NRE combinatorial correction (ignoring resolution effects) used as a baseline in the RMSE comparison)
- **MATLAB** (Development environment for ElemCor; MATLAB functions can be adapted into the simulation/benchmark pipeline)

## Examples

```
matlab -batch "elemcor('test_labelN_sim.xlsx','test_unlabelN_sim.xlsx','15N',140000,'Orbitrap')"  % then compute RMSE of enrichment vs 0.20
```

## Evaluation signals

- Simulated FAM vectors have length Nt+1, covering M+0 through M+Nt isotopologues for each metabolite
- Modified MDT and modified ULS yield significantly lower RMSE from the theoretical 20% enrichment than FAM-only, NRE, and original MDT/ULS implementations
- Output RMSE table contains one row per metabolite-method pair for all 24 metabolites and all correction methods
- Labeled and unlabeled simulated files have the same analytes and the same number of rows
- Processing of the example-size files completes within roughly a minute on a standard desktop CPU (README: ~10 s on a 2.6GHz Core i7)

## Limitations

- The validation data are simulated (XCalibur) plus a single-organism experiment (S. cerevisiae with 20.0% 15N ammonium), limiting generalization to other organisms and tracers
- ULS requires an additional unlabeled-sample measurement, adding experimental cost; MDT is recommended for low-mass compounds and ULS for high-mass compounds with relatively large spectral inaccuracy
- ElemCor was developed under MATLAB (2016b) and requires MATLAB Compiler Runtime; no software version, license terms, or public accession for validation datasets are stated
- The correction outcome is monotonically dependent on nominal resolution: at extremely low resolution ElemCor equals IsoCor; at extremely high resolution it performs no correction

## Evidence

- [readme] FAM vector length convention: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [readme] Simulated test file format and resolution: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
- [discussion] Non-negative regression for MDV: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [discussion] ULS deconvolution procedure: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [discussion] NRE baseline is IsoCor: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [readme] Resolution monotonicity: "Therefore, the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
