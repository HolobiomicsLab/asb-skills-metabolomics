---
name: isotope-natural-abundance-correction-matrix-construction
description: Use when correcting LC-MS isotope-labeling FAM data and you need the
  theoretical NRE correction matrix built from compound formula, isotope natural abundances,
  and tracer isotopic purity (e.g., 0.99 atomic for U-13C6 glucose).
when_to_use_negative:
- Data come from a high-resolving-power mass spectrometer (e.g., Orbitrap at 140,000
  or 280,000 nominal resolution) that resolves isotopologs with the same nominal mass
  — including all isotopologs in the correction matrix is then no longer justified
  and resolution-dependent (MDT/ULS) correction is preferred
- The selected tracer element is not present in the compound's formula — the algorithm
  will report an error (e.g., choosing C or S as tracer for 15N data where compounds
  lack C or S)
- The FAM data are already corrected MDVs or the analysis goal is flux fitting (INCA/FiatFlux)
  rather than isotope correction
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ElemCor
  role: Software implementing the correction-matrix approach; consumes tracer isotopic
    purity (Step 3) and tracer element (Step 4) to build the theoretical matrix and
    compute MDV/enrichment
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference NRE implementation whose purely combinatorial correction this matrix
    construction reproduces as the low-resolution baseline
- name: MATLAB
  role: Implementation environment (2016b) providing ElemCor functions adaptable into
    metabolic flux analysis pipelines such as FiatFlux and INCA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:320dcf410531902f000f98ff9369dd5e5711b5947e0ab7a9f50788d887ef09fb
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# isotope-natural-abundance-correction-matrix-construction

## Summary

Constructs the theoretical (non-resolution-effect, NRE) isotope correction matrix by combinatorial convolution of natural-abundance isotope distributions for all elements plus tracer isotopic impurity, as in IsoCor and as embedded in ElemCor's Step 3/4 parameters. It yields the (Nt+1)x(Nt+1) matrix that maps measured fractional abundances of measured isotopologs (FAM) to corrected mass distribution vectors (MDV).

## When to use

Use when correcting LC-MS isotope-labeling FAM data and you need the theoretical NRE correction matrix built from compound formula, isotope natural abundances, and tracer isotopic purity (e.g., 0.99 atomic for U-13C6 glucose).

## When NOT to use

- Data come from a high-resolving-power mass spectrometer (e.g., Orbitrap at 140,000 or 280,000 nominal resolution) that resolves isotopologs with the same nominal mass — including all isotopologs in the correction matrix is then no longer justified and resolution-dependent (MDT/ULS) correction is preferred
- The selected tracer element is not present in the compound's formula — the algorithm will report an error (e.g., choosing C or S as tracer for 15N data where compounds lack C or S)
- The FAM data are already corrected MDVs or the analysis goal is flux fitting (INCA/FiatFlux) rather than isotope correction

## Inputs

- Compound molecular formula specifying tracer atom count Nt and non-tracer elements (e.g., C6H12O6 with 13C as tracer)
- Natural-abundance isotope distributions for all elements in the formula
- Tracer isotopic purity of the nutrient (ElemCor Step 3 parameter, e.g., 0.99 atomic purity for U-13C6 glucose)
- FAM vector of length Nt+1 covering M+0 through M+Nt isotopologues (from XLSX input files such as test_labelN_sim.xlsx)

## Outputs

- C1 matrix: natural-abundance contribution of non-tracer elements
- C2 matrix: tracer-element natural-abundance contribution over Nt positions
- C3 matrix: tracer isotopic-impurity contribution
- Combined correction matrix C1*C2*C3 with dimensions (Nt+1)x(Nt+1)
- Corrected MDV and isotopic enrichment after non-negative linear regression and normalization

## How to apply

Parse the compound formula to count tracer atoms (Nt, e.g., 6 carbons in glucose for U-13C6) and enumerate non-tracer elements. Build C1 by multinomial convolution of natural-abundance isotope distributions across all non-tracer atoms, C2 by combinatorics over the Nt tracer-element positions (tracer natural abundance), and C3 from the tracer isotopic impurity specified in ElemCor Step 3 (e.g., 0.99 for U-13C6 glucose). Multiply C1, C2, and C3 in that order to obtain the total correction matrix of dimensions (Nt+1)x(Nt+1), matching a FAM vector covering M+0 through M+Nt. This NRE matrix is the IsoCor-equivalent baseline: it ignores resolution effects, so on high-resolving-power instruments it should be compared against or replaced by MDT/ULS correction. Corrected MDV is then obtained by non-negative linear regression of the matrix equation followed by normalization to sum 1.

## Related tools

- **ElemCor** (Software implementing the correction-matrix approach; consumes tracer isotopic purity (Step 3) and tracer element (Step 4) to build the theoretical matrix and compute MDV/enrichment) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Reference NRE implementation whose purely combinatorial correction this matrix construction reproduces as the low-resolution baseline)
- **MATLAB** (Implementation environment (2016b) providing ElemCor functions adaptable into metabolic flux analysis pipelines such as FiatFlux and INCA)

## Examples

```
C = C1*C2*C3; assert(isequal(size(C),[Nt+1, Nt+1])); % U-13C6 glucose: 0.99^6 ~= 0.94 molecular purity from 99% atomic purity
```

## Evaluation signals

- Resulting matrix must be (Nt+1)x(Nt+1); for U-13C6 glucose this is 7x7, matching a FAM vector of length Nt+1 covering M+0 through M+6
- Atomic-to-molecular purity verification: 0.99^6 must give ~0.94 molecular purity (probability all six carbons are labeled)
- Limit check: at extremely low instrument resolution the correction should be identical to IsoCor's NRE result; at extremely high resolution it should approach no correction
- After solving, MDV values must be non-negative and normalized to sum to 1
- Corrected enrichment must be reported for each compound/sample and stored in a new sheet of the input XLSX file

## Limitations

- The NRE (IsoCor-style) matrix ignores resolution effects and was shown to be significantly less accurate than resolution-dependent correction methods on high-resolution instruments
- Including all isotopologs in the correction matrix is not justified on modern high-resolving-power instruments that resolve same-nominal-mass isotopologs
- ElemCor depends on the proprietary MATLAB (2016b) environment / MATLAB Compiler Runtime, which may limit reproducibility

## Evidence

- [intro] FAM-to-MDV correction rationale: "the fractional abundance of measured isotopologs (FAM) collected from the instrument must be corrected to obtain MDV."
- [discussion] NRE baseline inferiority: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [intro] high-resolution limit on matrix composition: "including all isotopologs in the correction matrix is no longer justified"
- [methods] isotopic purity parameter location: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [readme] FAM vector dimensionality: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues."
