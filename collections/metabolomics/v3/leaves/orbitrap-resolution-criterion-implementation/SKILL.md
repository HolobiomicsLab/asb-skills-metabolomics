---
name: orbitrap-resolution-criterion-implementation
description: Use when correcting isotope-labeling LC-MS data (e.g. 13C, 15N, 2H, 18O,
  34S tracers) on a high-resolving-power Orbitrap, where same-nominal-mass isotopologs
  are partially resolvable and the correction matrix must be resolution-dependently
  zeroed instead of including all isotopologs as IsoCor does.
when_to_use_negative:
- When the instrument resolution is extremely low such that same-nominal-mass isotopologs
  cannot be resolved — there ElemCor/MDT is identical to IsoCor's combinatorial correction
  and the resolution criterion is moot
- When unlabeled-sample measurements are available and the compound is high-mass with
  relatively large spectral inaccuracy — use the ULS-based correction instead (MDT
  is recommended for low-mass compounds)
- When the selected tracer element is not present in a compound's formula — the README
  warns the algorithm will report an error (e.g. selecting C or S as tracer for 15N-labeled
  data)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ElemCor
  role: Provides the resolution-dependent correction matrix construction with the
    improved MDT isotope-inclusion criterion and MATLAB function adaptable to flux
    pipelines
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Supplies the mass-difference-theory criterion (Delta-M vs smallest resolvable
    mass difference) that determines which isotopologs are zeroed from the correction
    matrix
- name: IsoCor
  role: Baseline combinatorial correction without resolution effect (NRE) used as
    the comparison point; equivalent to ElemCor at extremely low resolution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:ce185963787a9d3e108b31f2942b1c187c6815af65aec1b3eda2a2b76355fc42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MDT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# orbitrap-resolution-criterion-implementation

## Summary

Implements the Orbitrap mass-difference-theory (MDT) criterion that decides which same-nominal-mass isotopologs are resolvable and thus excluded (zeroed) from the isotope correction matrix, including ElemCor's improved sum-of-mass-differences enumeration over isotope combinations.

## When to use

Use when correcting isotope-labeling LC-MS data (e.g. 13C, 15N, 2H, 18O, 34S tracers) on a high-resolving-power Orbitrap, where same-nominal-mass isotopologs are partially resolvable and the correction matrix must be resolution-dependently zeroed instead of including all isotopologs as IsoCor does.

## When NOT to use

- When the instrument resolution is extremely low such that same-nominal-mass isotopologs cannot be resolved — there ElemCor/MDT is identical to IsoCor's combinatorial correction and the resolution criterion is moot
- When unlabeled-sample measurements are available and the compound is high-mass with relatively large spectral inaccuracy — use the ULS-based correction instead (MDT is recommended for low-mass compounds)
- When the selected tracer element is not present in a compound's formula — the README warns the algorithm will report an error (e.g. selecting C or S as tracer for 15N-labeled data)

## Inputs

- Compound elemental formula (e.g. from the ElemCor test-data context: glutamine C5H10N2O3)
- Nominal instrument resolution and mass analyzer type (e.g. Orbitrap, R=100,000; test files at 140,000 and 280,000)
- Isotope mass differences for elements present in the formula (13C, 15N, 2H, 17O, 18O)
- Tracer element specification and tracer isotopic purity (Step 3/4 of the ElemCor workflow)

## Outputs

- Smallest resolvable Delta-M at the monoisotopic mass (e.g. ~2.07e-3 Da for glutamine at R=100,000)
- Per-isotope exclusion limits (counts retained vs zeroed in the correction matrix)
- Per-combination inclusion decisions for same-nominal-mass isotopologs (e.g. 2x17O + 1x18O retained as in Fig. 5b)
- Resolution-dependent MDT correction matrix rows for downstream non-negative linear regression to MDV

## How to apply

Load the compound formula (e.g. glutamine C5H10N2O3) and the nominal instrument resolution (e.g. 100,000 for Orbitrap). Compute the smallest resolvable mass difference with the Orbitrap criterion Delta-M >= 1.66*M^1.5/(R*sqrt(M_R)) evaluated at the monoisotopic mass — for glutamine at R=100,000 this is ~2.07e-3 Da. For each isotope present in the formula (13C, 15N, 2H, 17O, 18O), set the per-isotope exclusion limit to the largest count whose cumulative mass difference stays below the smallest resolvable Delta-M (glutamine reference: limit 2 for 17O, 0 for 15N, 2H, 18O). Then apply ElemCor's improvement over the original MDT (which was limited to tracers 13C, 2H, 15N): enumerate isotope combinations and sum their mass differences over the combination rather than excluding each isotope independently, so combinations like 2x17O + 1x18O that fit within the resolvable window are retained in the correction matrix. The resulting matrix feeds non-negative linear regression (Eq. 1) followed by normalization to obtain MDVs.

## Related tools

- **ElemCor** (Provides the resolution-dependent correction matrix construction with the improved MDT isotope-inclusion criterion and MATLAB function adaptable to flux pipelines) — https://github.com/4dsoftware/elemcor
- **MDT** (Supplies the mass-difference-theory criterion (Delta-M vs smallest resolvable mass difference) that determines which isotopologs are zeroed from the correction matrix)
- **IsoCor** (Baseline combinatorial correction without resolution effect (NRE) used as the comparison point; equivalent to ElemCor at extremely low resolution)

## Examples

```
dM_min = 1.66*M**1.5/(R*np.sqrt(M_R))  # glutamine M~146.045, R=100000 -> ~2.07e-3 Da
```

## Evaluation signals

- For glutamine (C5H10N2O3) at Orbitrap nominal resolution 100,000, the computed smallest resolvable Delta-M must be ~2.07e-3 Da
- Per-isotope exclusion limits at this resolution must reproduce the reference: 2 for 17O and 0 for 15N, 2H, and 18O
- The 2x17O + 1x18O combination must be included in the correction matrix under the improved sum-of-mass-differences criterion (Fig. 5b behavior), which the original per-isotope MDT criterion would drop
- Sanity check of monotonicity: at extremely low resolution the corrected result matches IsoCor; at extremely high resolution ElemCor performs no correction at all
- Corrected MDV vectors from non-negative regression must be normalized to sum 1 before enrichment reporting

## Limitations

- The original MDT isotope-exclusion criterion was limited to tracers 13C, 2H, and 15N; only ElemCor's modification extends the combination-wise treatment (ElemCor additionally allows 18O and 34S as tracers)
- Nominal instrument resolution is user-specified in Step 3 — an incorrect resolution or tracer selection yields wrong zeroing; README warns to 'MAKE SURE THE CORRECT TRACER IS SELECTED!'
- If the tracer element is absent from a compound's formula, the algorithm reports an error (e.g. 15N data analyzed with C or S selected as tracer)
- MDT assumes spectral accuracy sufficient for the mass-difference criterion; for high-mass compounds with relatively large spectral inaccuracy, ULS is recommended instead

## Evidence

- [abstract] Resolution-dependent matrix construction: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [intro] Motivation for zeroing same-nominal-mass isotopologs: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
- [methods] Resolution and tracer specification workflow: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [methods] Supported tracer elements for the criterion: "In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [readme] Monotonic resolution dependence of the correction: "the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [discussion] Output handling of corrected MDVs: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
