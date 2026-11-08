---
name: multinomial-isotopologue-distribution-computation
description: Use when correcting measured fractional abundances of isotopologs (FAM)
  to mass distribution vectors (MDV) and you need the theoretical natural-abundance
  + isotopic-impurity correction matrix for a compound of known formula and a specified
  tracer (e.g., U-13C6 glucose).
when_to_use_negative:
- On high-resolving-power mass spectrometers (e.g., Orbitrap at 140,000–280,000 nominal
  resolution) that can resolve isotopologs of the same nominal mass — the NRE matrix
  is significantly less accurate than resolution-dependent corrections (MDT/ULS).
- When the selected tracer element is not present in the compound's formula — ElemCor's
  algorithm reports an error (e.g., selecting C or S as tracer for 15N data).
- When the goal is tracer-analysis correction on unlabeled samples for resolution-effect
  estimation rather than theoretical matrix construction (use ULS deconvolution instead).
tools:
- name: ElemCor
  role: Software implementing the correction-matrix framework; NRE (IsoCor-style)
    correction is its resolution-effect-free baseline and Step 3 supplies the isotopic-purity
    parameter used for matrix C3
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference combinatorial correction method that computes the same natural-abundance/impurity
    matrix without resolution effects
- name: MATLAB
  role: Environment in which ElemCor (developed under MATLAB 2016b) provides functions
    adaptable into flux-analysis pipelines such as FiatFlux and INCA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:dcd42e3d79dfa5edc7c801a11817a166c0fd75d4d9751d77fb3a44de7a1d30e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# multinomial-isotopologue-distribution-computation

## Summary

Constructs the theoretical (non-resolution-effect, NRE) isotope correction matrix C1·C2·C3 for stable-isotope labeling LC-MS data by multinomial convolution of natural-abundance isotope distributions over non-tracer atoms, combinatorics over the Nt tracer positions, and the tracer's nutrient isotopic purity — the combinatorial baseline used by IsoCor and as the NRE comparator in ElemCor.

## When to use

Use when correcting measured fractional abundances of isotopologs (FAM) to mass distribution vectors (MDV) and you need the theoretical natural-abundance + isotopic-impurity correction matrix for a compound of known formula and a specified tracer (e.g., U-13C6 glucose).

## When NOT to use

- On high-resolving-power mass spectrometers (e.g., Orbitrap at 140,000–280,000 nominal resolution) that can resolve isotopologs of the same nominal mass — the NRE matrix is significantly less accurate than resolution-dependent corrections (MDT/ULS).
- When the selected tracer element is not present in the compound's formula — ElemCor's algorithm reports an error (e.g., selecting C or S as tracer for 15N data).
- When the goal is tracer-analysis correction on unlabeled samples for resolution-effect estimation rather than theoretical matrix construction (use ULS deconvolution instead).

## Inputs

- Compound molecular formula (to count Nt tracer atoms and enumerate non-tracer elements)
- Isotope natural-abundance distributions for each element
- Tracer element identity (13C, 2H, 15N, 18O, or 34S in ElemCor)
- Nutrient/tracer atomic isotopic purity (ElemCor Step 3 parameter, e.g., 0.99 for U-13C6 glucose)
- Measured FAM vector (length Nt+1 covering M+0..M+Nt isotopologues), e.g., from an XLSX input file

## Outputs

- Total theoretical correction matrix of dimensions (Nt+1)×(Nt+1) assembled as C1·C2·C3
- Corrected MDV (mass distribution vector) via non-negative linear regression and normalization to sum 1
- Isotopic enrichment per compound

## How to apply

Parse the compound formula to count tracer atoms Nt (e.g., 6 carbons in glucose for U-13C6) and enumerate the non-tracer elements. Build C1 for non-tracer elements by multinomial convolution of their natural-abundance isotope distributions across all non-tracer atoms; build C2 for tracer-element natural abundance via combinatorics over the Nt tracer positions; build C3 for tracer isotopic impurity from the nutrient isotopic purity specified in ElemCor's Step 3 (e.g., 0.99 for U-13C6 glucose). Multiply C1, C2, and C3 in that order to form the total (Nt+1)×(Nt+1) correction matrix matching the FAM vector length covering M+0 through M+Nt. Then apply non-negative linear regression of the correction-matrix equation followed by normalization (sum to 1) to obtain MDV and isotopic enrichment. Note this NRE-style matrix is the resolution-independent baseline: it ignores mass-difference-theory resolution effects, so it is exact only at very low instrument resolution where ElemCor is identical to IsoCor.

## Related tools

- **ElemCor** (Software implementing the correction-matrix framework; NRE (IsoCor-style) correction is its resolution-effect-free baseline and Step 3 supplies the isotopic-purity parameter used for matrix C3) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Reference combinatorial correction method that computes the same natural-abundance/impurity matrix without resolution effects)
- **MATLAB** (Environment in which ElemCor (developed under MATLAB 2016b) provides functions adaptable into flux-analysis pipelines such as FiatFlux and INCA)

## Examples

```
0.99^6 % ElemCor U-13C6 glucose validation: 99% atomic purity -> ~0.94 molecular purity, then solve C1*C2*C3 * MDV_true = FAM by non-negative regression
```

## Evaluation signals

- Resulting matrix dimensions are exactly (Nt+1)×(Nt+1) for the specified compound (e.g., 7×7 for U-13C6 glucose covering M+0..M+6).
- Molecular-purity check: 0.99^6 ≈ 0.94 must be reproduced for U-13C6 glucose at 99% atomic purity.
- Corrected MDV entries are non-negative and sum to 1 after the normalization step.
- In the extremely-low-resolution limit, results of this NRE matrix coincide with IsoCor's combinatorial correction (ElemCor = IsoCor at zero resolution effect).
- ElemCor runs to completion only when the selected tracer element is present in each compound's formula; a run error indicates wrong tracer selection rather than matrix failure.

## Limitations

- The NRE matrix ignores resolution effects, which are significant on modern high-resolving-power instruments that can resolve isotopologs of the same nominal mass; correction amount in ElemCor is monotonically dependent on instrument resolution.
- Tracer isotopic impurity is propagated from atomic purity (e.g., 0.99 atomic → ~0.94 molecular for 6 tracer atoms), so high tracer-atom counts amplify impurity effects.
- Existing implementations of natural-abundance corrections had mathematical defects that ElemCor corrects; older schemes (e.g., IsoCor) may not match.
- FAM vectors may have length other than Nt+1; ElemCor handles this, but naive matrix construction assumes length Nt+1.

## Evidence

- [intro] FAM must be corrected for naturally occurring isotopes and tracer isotopic impurity to obtain MDV: [evidence span withheld — anchor status: not_found]
- [discussion] correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [readme] In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified.: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [readme] IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor.: "IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor."
