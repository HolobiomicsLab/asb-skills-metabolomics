---
name: coenzyme-a-fractional-abundance-simulation
description: Use when benchmarking isotope-correction methods (ElemCor MDT/ULS, original
  ULS, NRE, FAM) on simulated or measured FAM for large sulfur- or nitrogen-containing
  metabolites at high nominal instrument resolution, or when validating 34S/13C correction
  accuracy for CoA-pathway metabolites.
when_to_use_negative:
- The compound set contains only low-mass metabolites for which MDT correction is
  recommended for cost efficiency and ULS is unnecessary
- The instrument operates at very low nominal resolution where ElemCor becomes identical
  to IsoCor's combinatorial correction, making the resolution-dependent simulation
  uninformative
- The tracer element is not present in the compound formula (e.g. selecting C or S
  for a compound lacking those atoms causes the algorithm to error)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ElemCor
  role: Runs MDT and modified ULS corrections on the simulated CoA metabolite FAM
    at resolution 280,000 and outputs MDVs and enrichments
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Platform in which ElemCor was developed (MATLAB 2016b); MATLAB functions can
    be incorporated into downstream flux-analysis pipelines such as FiatFlux or INCA
- name: MDT
  role: Mass-difference-theory-based correction mode used within ElemCor for low-mass
    compounds; for CoA metabolites ULS-based correction is preferred, but MDT is used
    as a comparison method where the tracer is supported
- name: ULS
  role: Unlabeled-sample-based correction mode within ElemCor, including the original
    defective ULS implementation and the improved version, used to show the accuracy
    gap for 13C and 34S on CoA metabolites
- name: IsoCor
  role: Combinatorial no-resolution-effect (NRE) correction used as a baseline comparison
    showing that resolution-ignoring correction is less accurate than ElemCor for
    high-mass metabolites
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:090a76c0df52fde6ff3563a992030e4e90171a1c947b76cf27b85592c1f7b4ee
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# coenzyme-a-fractional-abundance-simulation

## Summary

Simulates fractional abundance (FAM) data for high-mass coenzyme A derivatives (coenzyme A, acetyl-CoA, succinyl-CoA, HMG-CoA) under 13C, 15N, and 34S tracers at nominal Orbitrap resolution 280,000, to benchmark ElemCor's MDT/ULS corrections against original ULS, NRE, and no correction. This skill reproduces the paper's large-metabolite validation (Fig. 2a-c) showing that the original ULS method is excellent for 15N but much less accurate for 13C and 34S, whereas ElemCor resolves both.

## When to use

Use when benchmarking isotope-correction methods (ElemCor MDT/ULS, original ULS, NRE, FAM) on simulated or measured FAM for large sulfur- or nitrogen-containing metabolites at high nominal instrument resolution, or when validating 34S/13C correction accuracy for CoA-pathway metabolites.

## When NOT to use

- The compound set contains only low-mass metabolites for which MDT correction is recommended for cost efficiency and ULS is unnecessary
- The instrument operates at very low nominal resolution where ElemCor becomes identical to IsoCor's combinatorial correction, making the resolution-dependent simulation uninformative
- The tracer element is not present in the compound formula (e.g. selecting C or S for a compound lacking those atoms causes the algorithm to error)

## Inputs

- Simulated or measured fractional abundance (FAM) column vectors for coenzyme A, acetyl-CoA, succinyl-CoA, and HMG-CoA at nominal resolution 280,000, stored in XLSX files in ElemCor's standard format (one column per sample)
- ElemCor package test data: test_labelN_sim.xlsx, test_labelS_sim.xlsx, test_unlabelN_sim.xlsx, test_unlabelS_sim.xlsx
- ElemCor executable built from elemcor.prj or the pre-compiled ElemCor.exe
- Nominal instrument resolution (280,000), tracer element (13C, 15N, or 34S), mass analyzer (Orbitrap), and isotopic purity of the nutrient

## Outputs

- Per-compound, per-sample MDV values stored as a new sheet in the original XLSX input file
- Isotopic enrichment values for each CoA metabolite under each tracer and correction method
- RMSE comparison table (e.g. rmse_table.csv) quantifying accuracy of ElemCor MDT, ElemCor ULS, original ULS, no-resolution-effect (NRE), and no correction against theoretical enrichment

## How to apply

Generate simulated FAM column vectors for the four CoA metabolites with known theoretical enrichment (e.g. 20%) using the XCalibur/Orbitrap simulation convention, one column per sample, at nominal resolution 280,000 as in the package's test files. Load labeled and unlabeled test data (e.g. test_labelS_sim.xlsx, test_unlabelS_sim.xlsx) into ElemCor, select 34S or 15N as tracer, choose the Orbitrap mass analyzer, specify resolution 280,000, and run MDT and modified ULS corrections; original MDT does not support 34S tracers. Compute non-negative linear regression of the correction-matrix equation followed by normalization to a sum of 1 to obtain MDVs, then isotopic enrichment per compound. Compute RMSE of corrected enrichment against the theoretical enrichment for each compound-tracer-method combination. The expected ordering matches the paper: original ULS is excellent for 15N but remarkably lower for 13C and 34S, while ElemCor's corrected methods remain accurate across all three tracers.

## Related tools

- **ElemCor** (Runs MDT and modified ULS corrections on the simulated CoA metabolite FAM at resolution 280,000 and outputs MDVs and enrichments) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Platform in which ElemCor was developed (MATLAB 2016b); MATLAB functions can be incorporated into downstream flux-analysis pipelines such as FiatFlux or INCA)
- **MDT** (Mass-difference-theory-based correction mode used within ElemCor for low-mass compounds; for CoA metabolites ULS-based correction is preferred, but MDT is used as a comparison method where the tracer is supported)
- **ULS** (Unlabeled-sample-based correction mode within ElemCor, including the original defective ULS implementation and the improved version, used to show the accuracy gap for 13C and 34S on CoA metabolites)
- **IsoCor** (Combinatorial no-resolution-effect (NRE) correction used as a baseline comparison showing that resolution-ignoring correction is less accurate than ElemCor for high-mass metabolites)

## Examples

```
# Build / run ElemCor (no MATLAB required) with the provided test files:
#   elemcor.prj -> ElemCor.exe
# Load: test_labelS_sim.xlsx / test_unlabelS_sim.xlsx
# Select tracer: 34S, analyzer: Orbitrap, resolution: 280000
# Output: MDV + enrichment sheets written back to the same .xlsx; compute RMSE vs 20% theoretical enrichment per method
```

## Evaluation signals

- Modified ElemCor MDT and ULS should show lower enrichment RMSE than original ULS for 13C and 34S tracers on all four CoA metabolites
- Original ULS should be highly accurate for 15N (matching the paper's finding that it is excellent for 15N) but show markedly higher RMSE for 13C and 34S
- NRE (IsoCor-style) correction should be less accurate than ElemCor for all three tracers at resolution 280,000, reflecting its inability to account for resolution effect
- Corrected MDVs must be normalized to sum to 1 after non-negative linear regression of the correction-matrix equation
- Output MDV and enrichment values should appear as additional sheets in the original XLSX input file, matching ElemCor's documented behavior

## Limitations

- Original ULS has a mathematical defect that makes it less accurate for tracers whose most abundant isotope has lower natural abundance, such as 32S (95.0%) versus 14N (99.6%); ElemCor's modified ULS resolves this
- Original MDT does not support 34S as a tracer, limiting method comparison for sulfur-labeled experiments
- ULS requires additional unlabeled-sample measurements, adding experimental cost; MDT is recommended for small metabolites where this is impractical
- ElemCor was developed under MATLAB 2016b and relies on MATLAB Compiler Runtime; source-code changes require rebuilding via elemcor.prj
- If results do not appear within a minute, the wrong tracer element was likely selected (e.g. 15N data with C or S selected), because the algorithm errors when the tracer element is absent from a compound
- Validation in the paper was performed on simulated data plus a single experimental system (S. cerevisiae grown with 20.0% 15N ammonium); extrapolation to other organisms or tracers should be re-validated

## Evidence

- [methods] ElemCor supports 34S correction at high resolution and CoA metabolite accuracy: "In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [discussion] ULS correction relies on unlabeled-sample deconvolution: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [discussion] ElemCor MDVs computed by non-negative linear regression then normalized: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [readme] Standard test-file format: FAM columns per sample at nominal resolutions 140,000 and 280,000: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
- [readme] ElemCor's correction is monotonically dependent on instrument resolution, spanning IsoCor-like behavior at low resolution to no correction at high resolution: "The amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [readme] Corrected MDVs and enrichments are written back as new sheets in the original input XLSX: "They are automatically saved into additional sheets in the original XLSX file."
