---
name: elemental-formula-tracer-atom-enumeration
description: Use when correcting LC-MS isotope-labeling FAM/MDV data for natural abundance
  and tracer impurity at low resolution, or when building the NRE baseline against
  which resolution-dependent corrections (MDT/ULS) are compared.
when_to_use_negative:
- High-resolving-power MS (e.g., Orbitrap at 140,000 or 280,000 nominal resolution)
  can resolve isotopologs of the same nominal mass, so including all isotopologs in
  a purely combinatorial matrix is 'no longer justified' — use a resolution-dependent
  correction (MDT/ULS) instead
- Compounds whose formula does not contain the selected tracer element (e.g., selecting
  C or S as tracer for 15N-labeled data) — the ElemCor algorithm reports an error
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Software implementing the correction-matrix framework (with MDT/ULS resolution
    handling); Step 3 supplies isotopic purity of nutrient, Step 4 selects the tracer
    element feeding C2/C3 construction
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Combinatorial NRE correction method that this matrix construction reproduces;
    serves as the low-resolution limit and accuracy baseline
- name: MATLAB
  role: Development/host environment; ElemCor MATLAB functions can be embedded in
    flux analysis pipelines (FiatFlux, INCA)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:597e3bd034bb9c0d3c15304d00a4899ca90d5e77c30bfa2f1920c40c67df8bb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# elemental-formula-tracer-atom-enumeration

## Summary

Constructs the theoretical (non-resolution-effect, NRE) isotope correction matrix by combinatorial convolution of natural-abundance isotope distributions (C1), tracer-element natural abundance (C2), and tracer isotopic impurity (C3), as used by IsoCor and as the NRE baseline inside ElemCor.

## When to use

Use when correcting LC-MS isotope-labeling FAM/MDV data for natural abundance and tracer impurity at low resolution, or when building the NRE baseline against which resolution-dependent corrections (MDT/ULS) are compared.

## When NOT to use

- High-resolving-power MS (e.g., Orbitrap at 140,000 or 280,000 nominal resolution) can resolve isotopologs of the same nominal mass, so including all isotopologs in a purely combinatorial matrix is 'no longer justified' — use a resolution-dependent correction (MDT/ULS) instead
- Compounds whose formula does not contain the selected tracer element (e.g., selecting C or S as tracer for 15N-labeled data) — the ElemCor algorithm reports an error

## Inputs

- Elemental formula of each compound (to enumerate Nt tracer atoms and non-tracer elements)
- FAM (fractional abundance of measured isotopologues) vector, typically length Nt+1 covering M+0..M+Nt
- Natural-abundance isotope distributions for each non-tracer element
- Nutrient isotopic purity (atomic fraction, e.g., 0.99 for U-13C6 glucose) from ElemCor Step 3
- Tracer element selection (13C, 2H, 15N, 18O, or 34S) from ElemCor Step 4

## Outputs

- Total theoretical correction matrix C = C1 x C2 x C3 with dimensions (Nt+1) x (Nt+1)
- Molecular (all-position) tracer purity implied by atomic purity, e.g., 0.99^6 ≈ 0.94 for U-13C6 glucose

## How to apply

Parse the compound formula to count tracer atoms (Nt, e.g., 6 carbons in U-13C6 glucose) and enumerate all non-tracer elements. Build C1 by multinomial convolution of natural-abundance isotope distributions across all non-tracer atoms; build C2 by combinatorics over the Nt tracer positions (tracer-element natural abundance); build C3 from the nutrient isotopic purity specified in Step 3 of ElemCor (e.g., 0.99 atomic purity for U-13C6 glucose). Multiply C1, C2, and C3 in that order to form the (Nt+1)x(Nt+1) total correction matrix matching the FAM vector length covering M+0 through M+Nt. Note the rationale: at low instrument resolution IsoCor-style combinatorial correction is appropriate — the README states ElemCor 'is identical to IsoCor' at extremely low resolution, and IsoCor's NRE correction was 'significantly less accurate' than resolution-aware methods on high-resolution data. Apply the matrix by non-negative linear regression of the correction equation followed by normalization to sum 1 to obtain MDV and isotopic enrichment.

## Related tools

- **ElemCor** (Software implementing the correction-matrix framework (with MDT/ULS resolution handling); Step 3 supplies isotopic purity of nutrient, Step 4 selects the tracer element feeding C2/C3 construction) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Combinatorial NRE correction method that this matrix construction reproduces; serves as the low-resolution limit and accuracy baseline)
- **MATLAB** (Development/host environment; ElemCor MATLAB functions can be embedded in flux analysis pipelines (FiatFlux, INCA))

## Examples

```
In ElemCor GUI: load test_labelN_sim.xlsx (Step 1), set isotopic purity 0.99 and resolution 140000 (Step 3), select 15N as tracer (Step 4), then analyze (Step 6)
```

## Evaluation signals

- Combined matrix C must be exactly (Nt+1) x (Nt+1) for the compound, matching the FAM vector length covering M+0 through M+Nt
- Atomic-to-molecular purity check: U-13C6 glucose at 99% atomic purity must yield ~94% probability that all six carbons are labeled (0.99^6 ≈ 0.94)
- Low-resolution limit check: at extremely low instrument resolution the corrected MDV should match IsoCor (NRE) output; at extremely high resolution ElemCor performs essentially no correction
- Applying the matrix via non-negative linear regression followed by normalization must yield an MDV whose elements sum to 1
- MDV after correction should be stored (ElemCor writes corrected MDVs to a new sheet of the input XLSX) and enrichment recalculated per compound in Step 6

## Limitations

- NRE (IsoCor-style) correction ignoring resolution effects was significantly less accurate than resolution-aware methods on high-resolution data
- Requires correct tracer selection; mismatched tracer (e.g., C or S selected for 15N data) causes the algorithm to fail on compounds lacking that element
- Molecular purity is not equal to atomic purity: 99% atomic U-13C6 glucose gives only ~94% fully labeled molecules (0.99^6)

## Evidence

- [readme] FAM vector length: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [readme] IsoCor low-resolution identity: "IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor."
- [discussion] NRE accuracy: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [intro] FAM to MDV correction: "the fractional abundance of measured isotopologs (FAM) collected from the instrument must be corrected to obtain MDV."
- [discussion] non-negative regression: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [intro] isotopolog inclusion no longer justified: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
