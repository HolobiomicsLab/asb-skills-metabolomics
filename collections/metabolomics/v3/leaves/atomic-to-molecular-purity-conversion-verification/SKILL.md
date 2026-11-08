---
name: atomic-to-molecular-purity-conversion-verification
description: Use when reconstructing or validating the theoretical correction matrix
  for LC-MS isotope labeling data — e.g., checking that an (Nt+1)x(Nt+1) matrix correctly
  converts nutrient atomic purity (Step 3 parameter, e.g., 0.99) into molecular purity
  for a tracer like U-13C6 glucose.
when_to_use_negative:
- High-resolution instruments where isotopologs of the same nominal mass are resolved
  — including all isotopologs in the correction matrix is no longer justified; use
  resolution-dependent correction (ElemCor MDT/ULS) instead
- The tracer element is absent from the compound formula — the algorithm errors if
  the selected tracer (e.g., C or S) is not present, so the matrix cannot be built
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ElemCor
  role: Software whose Step 3 isotopic purity parameter and correction-matrix framework
    the constructed C1/C2/C3 matrix must reproduce (NRE baseline)
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference combinatorial correction without resolution effect (NRE) that this
    matrix construction matches
- name: MATLAB
  role: Environment for implementing the multinomial convolutions and matrix products;
    ElemCor provides a MATLAB function adaptable to flux analysis pipelines
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory approach that extends the NRE matrix with resolution
    dependence — used as contrast for the theoretical-only construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:dc7ffd30bc08ce22b90d675646485ec4767e33493d9732a3f26e2319abc01ed3
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# atomic-to-molecular-purity-conversion-verification

## Summary

Verification skill for the theoretical (non-resolution-effect, NRE) isotope correction matrix used by ElemCor/IsoCor: build C1 (non-tracer natural abundance), C2 (tracer-element natural abundance), and C3 (tracer isotopic impurity) matrices by combinatorial calculation, multiply them, and confirm the atomic-to-molecular purity relation (0.99^6 ≈ 0.94 for U-13C6 glucose).

## When to use

Use when reconstructing or validating the theoretical correction matrix for LC-MS isotope labeling data — e.g., checking that an (Nt+1)x(Nt+1) matrix correctly converts nutrient atomic purity (Step 3 parameter, e.g., 0.99) into molecular purity for a tracer like U-13C6 glucose.

## When NOT to use

- High-resolution instruments where isotopologs of the same nominal mass are resolved — including all isotopologs in the correction matrix is no longer justified; use resolution-dependent correction (ElemCor MDT/ULS) instead
- The tracer element is absent from the compound formula — the algorithm errors if the selected tracer (e.g., C or S) is not present, so the matrix cannot be built

## Inputs

- Compound molecular formula (e.g., C6H12O6 glucose) with tracer element and tracer-atom count Nt
- Nutrient/tracer isotopic purity parameter from ElemCor Step 3 (e.g., 0.99 atomic purity for U-13C6 glucose)
- Natural-abundance isotope distributions for tracer and non-tracer elements (13C, 2H, 15N, 18O, 34S supported as tracers)
- FAM (fractional abundance of measured isotopologues) vector of length Nt+1 covering M+0..M+Nt

## Outputs

- Total theoretical correction matrix of dimensions (Nt+1) x (Nt+1) = C1 * C2 * C3
- Verified molecular purity value for the tracer nutrient (e.g., ~0.94 molecular purity from 0.99 atomic purity for U-13C6 glucose)

## How to apply

Parse the compound formula to count tracer atoms (Nt, e.g., 6 carbons in glucose for U-13C6) and enumerate non-tracer elements. Build C1 by multinomial convolution of natural-abundance isotope distributions across all non-tracer atoms; build C2 via combinatorics over the Nt tracer positions; build C3 from the nutrient isotopic purity specified in ElemCor's Step 3 (e.g., 0.99 for U-13C6 glucose). Multiply C1, C2, and C3 in that order to form the total correction matrix of dimensions (Nt+1)x(Nt+1), matching the FAM vector length covering M+0 through M+Nt isotopologues (ElemCor can also handle FAM lengths other than Nt+1). Verify the atomic-to-molecular purity relation: at 99% atomic purity, the probability all six carbons are labeled is 0.99^6 ≈ 0.94. This NRE-style matrix corresponds to IsoCor's combinatorial approach; it ignores resolution effects, which is the correct baseline only when instrument resolution effects are deliberately excluded.

## Related tools

- **ElemCor** (Software whose Step 3 isotopic purity parameter and correction-matrix framework the constructed C1/C2/C3 matrix must reproduce (NRE baseline)) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Reference combinatorial correction without resolution effect (NRE) that this matrix construction matches)
- **MATLAB** (Environment for implementing the multinomial convolutions and matrix products; ElemCor provides a MATLAB function adaptable to flux analysis pipelines) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory approach that extends the NRE matrix with resolution dependence — used as contrast for the theoretical-only construction)

## Examples

```
molecular_purity = 0.99^6 % ≈ 0.9415: probability all 6 carbons labeled for U-13C6 glucose at 99% atomic purity
```

## Evaluation signals

- Total matrix product C1*C2*C3 must have dimensions (Nt+1) x (Nt+1) for the specified compound (6x6 for glucose U-13C6)
- Glucose verification must reproduce ~0.94 molecular purity from 99% atomic purity (0.99^6 ≈ 0.9415)
- Matrix rows/columns must correspond to M+0 through M+Nt isotopologues, matching the FAM vector length
- At extremely low instrument resolution the NRE matrix must coincide with full IsoCor correction; at extremely high resolution it should apply no correction
- Corrected MDV obtained by non-negative linear regression against the matrix followed by normalization must sum to 1

## Limitations

- The theoretical matrix ignores resolution effects; on high-resolving-power MS that resolve same-nominal-mass isotopologs, NRE correction (as in IsoCor) was significantly less accurate than resolution-dependent methods
- FAM lengths other than Nt+1 require ElemCor's extended handling; a naive (Nt+1)x(Nt+1) matrix will not match
- Matrix multiplication order (C1, C2, C3) matters; applying C3 from atomic purity without the ^Nt exponent conflates atomic and molecular purity

## Evidence

- [methods] In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified.: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [readme] FAM length rule: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [discussion] NRE accuracy caveat: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [intro] Isotopolog inclusion caveat: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
