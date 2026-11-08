---
name: tracer-isotopic-impurity-matrix-parameterization
description: Use when correcting fractional abundances of measured isotopologs (FAM)
  for natural abundance and tracer isotopic impurity without resolution effects, e.g.
  building a baseline correction matrix for a compound with Nt tracer atoms.
when_to_use_negative:
- When the instrument's high resolving power can resolve isotopologs with the same
  nominal mass — including all isotopologs in the correction matrix is then no longer
  justified, and resolution-dependent methods (MDT/ULS) are preferred.
- When FAM data come from an unlabeled sample so tracer-impurity combinatorics are
  unnecessary — deconvolution of the unlabeled-sample FAM supplies the correction
  directly.
- When the selected tracer element is absent from the compound formula — the algorithm
  will report an error, e.g. selecting C or S as tracer for 15N-labeled data.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Implements the NRE correction matrix (C1, C2, C3) construction and applies
    it to FAM data to yield MDV and enrichment
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference combinatorial NRE correction method whose matrix construction ElemCor's
    baseline matches
- name: MATLAB
  role: Environment in which ElemCor's correction functions are provided and adaptable
    to flux analysis pipelines (e.g. INCA, FiatFlux)
- name: MDT
  role: Mass-difference-theory resolution-dependent alternative to the NRE matrix
    baseline within ElemCor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:a37d9df2f4a73cf21bba0280e0426b142e5d7c3fff52231694ceb45ff7f801fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tracer-isotopic-impurity-matrix-parameterization

## Summary

Constructs the theoretical natural-abundance/isotopic-impurity correction matrix (C1 x C2 x C3) for isotope-labeling correction by combinatorial convolution of isotope distributions over non-tracer elements, tracer-element natural abundance, and nutrient isotopic purity, as implemented in ElemCor's non-resolution-effect (NRE) baseline.

## When to use

Use when correcting fractional abundances of measured isotopologs (FAM) for natural abundance and tracer isotopic impurity without resolution effects, e.g. building a baseline correction matrix for a compound with Nt tracer atoms.

## When NOT to use

- When the instrument's high resolving power can resolve isotopologs with the same nominal mass — including all isotopologs in the correction matrix is then no longer justified, and resolution-dependent methods (MDT/ULS) are preferred.
- When FAM data come from an unlabeled sample so tracer-impurity combinatorics are unnecessary — deconvolution of the unlabeled-sample FAM supplies the correction directly.
- When the selected tracer element is absent from the compound formula — the algorithm will report an error, e.g. selecting C or S as tracer for 15N-labeled data.

## Inputs

- Compound molecular formula (to count Nt tracer atoms and enumerate non-tracer elements)
- Tracer element specification (13C, 2H, 15N, 18O, or 34S in ElemCor)
- Nutrient isotopic purity (Step 3 parameter, e.g. 0.99 for U-13C6 glucose)
- Natural-abundance isotope distributions for tracer and non-tracer elements
- FAM (fractional abundance of measured isotopologs) vector of length Nt+1

## Outputs

- Total theoretical correction matrix C = C1 x C2 x C3 of dimensions (Nt+1)x(Nt+1)
- Corrected MDV (mass distribution vector) after applying the matrix via non-negative linear regression of Eq. (1) followed by normalization to sum 1

## How to apply

Parse the compound formula to count tracer atoms (Nt, e.g. 6 carbons for U-13C6 glucose) and enumerate non-tracer elements, then build C1 by multinomial convolution of natural-abundance isotope distributions across all non-tracer atoms. Build C2 by combinatorics over the Nt tracer positions to capture tracer-element natural abundance, and C3 from the nutrient isotopic purity specified in Step 3 of ElemCor (e.g. 0.99 for U-13C6 glucose). Multiply C1, C2, and C3 in that order to form the total (Nt+1)x(Nt+1) correction matrix whose dimension matches the FAM vector covering M+0 through M+Nt isotopologues; ElemCor can also handle FAM lengths other than Nt+1. Verify the atomic-to-molecular purity relation by checking that 99% atomic purity for U-13C6 glucose yields ~0.94 molecular purity (0.99^6), since all six carbons must be labeled.

## Related tools

- **ElemCor** (Implements the NRE correction matrix (C1, C2, C3) construction and applies it to FAM data to yield MDV and enrichment) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Reference combinatorial NRE correction method whose matrix construction ElemCor's baseline matches)
- **MATLAB** (Environment in which ElemCor's correction functions are provided and adaptable to flux analysis pipelines (e.g. INCA, FiatFlux))
- **MDT** (Mass-difference-theory resolution-dependent alternative to the NRE matrix baseline within ElemCor)

## Evaluation signals

- Total correction matrix must be (Nt+1)x(Nt+1) for the specified compound, matching the FAM vector length covering M+0 through M+Nt isotopologues.
- For U-13C6 glucose at 99% atomic purity, the computed probability that all six carbons are labeled must be ~0.94 (0.99^6), verifying the atomic-to-molecular purity relation.
- Resulting MDV must sum to 1 after non-negative linear regression of the correction matrix equation followed by normalization.
- Result must match IsoCor's NRE output when resolution is extremely low (ElemCor is identical to IsoCor in that limit).

## Limitations

- NRE correction (as in IsoCor) was significantly less accurate than resolution-dependent methods on high-resolving-power instruments such as Orbitraps.
- The atomic-to-molecular purity gap grows with Nt: 99% atomic purity yields only ~94% molecular purity for a 6-carbon tracer, so even 'pure' tracers inject impurity signal at high Nt.
- ElemCor requires the tracer element to be present in the compound formula and the correct tracer to be selected, or the algorithm errors.
- ElemCor was developed under a proprietary MATLAB (2016b) environment, which may limit reproducibility.

## Evidence

- [methods] Theoretical correction matrix construction: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [methods] Isotopic impurity parameter: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [discussion] MDV normalization: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [readme] FAM dimension: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [discussion] NRE vs IsoCor: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
