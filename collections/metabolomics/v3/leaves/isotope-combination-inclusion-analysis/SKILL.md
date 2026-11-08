---
name: isotope-combination-inclusion-analysis
description: Use when correcting FAM to MDV on a high-resolution instrument (e.g.
  Orbitrap) and you must determine which same-nominal-mass isotopolog combinations
  the correction matrix can exclude without losing accuracy.
when_to_use_negative:
- Instrument resolution is extremely low or extremely high — ElemCor collapses to
  IsoCor-style correction at very low resolution and performs no correction at all
  at very high resolution.
- The tracer element is not present in the compound's formula — the algorithm reports
  an error (e.g. selecting C or S as tracer when 15N data are loaded).
- Only nominal-mass isotopologs matter because the instrument cannot resolve same-nominal-mass
  isotopologs — then all isotopologs should be included in the correction matrix as
  in IsoCor.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Software that implements the resolution-dependent, combination-summing correction
    matrix construction based on MDT
  repo: github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory providing the resolvable-Delta-M criterion that drives
    isotope exclusion/inclusion
  repo: github.com/4dsoftware/elemcor
- name: IsoCor
  role: Baseline correction method that does not consider resolution effects; used
    for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:1bc00d03d50456b8dbb6383778272301df23fd8e555b94b35bbf5727a3920a27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemCor
  - IsoCor
  - MDT
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# isotope-combination-inclusion-analysis

## Summary

Decides which same-nominal-mass isotopolog combinations must be included in a correction matrix as a function of instrument resolving power, using mass difference theory (MDT) as implemented and improved in ElemCor.

## When to use

Use when correcting FAM to MDV on a high-resolution instrument (e.g. Orbitrap) and you must determine which same-nominal-mass isotopolog combinations the correction matrix can exclude without losing accuracy.

## When NOT to use

- Instrument resolution is extremely low or extremely high — ElemCor collapses to IsoCor-style correction at very low resolution and performs no correction at all at very high resolution.
- The tracer element is not present in the compound's formula — the algorithm reports an error (e.g. selecting C or S as tracer when 15N data are loaded).
- Only nominal-mass isotopologs matter because the instrument cannot resolve same-nominal-mass isotopologs — then all isotopologs should be included in the correction matrix as in IsoCor.

## Inputs

- Molecular formula of the compound (e.g. glutamine C5H10N2O3)
- Nominal instrument resolution (e.g. 100,000 for Orbitrap)
- Mass analyzer type (Step 5 in the ElemCor workflow)
- Tracer element selected in Step 4 (13C, 2H, 15N, 18O, or 34S)

## Outputs

- Smallest resolvable Delta-M at the monoisotopic mass
- Per-isotope exclusion limits (e.g. 2 for 17O, 0 for 15N/2H/18O for glutamine at 100,000)
- Per-combination inclusion decisions (e.g. include 2x17O + 1x18O for glutamine at 100,000)
- Correction matrix that contains only the justified isotopolog combinations

## How to apply

Load the compound formula and the nominal instrument resolution specified in the ElemCor workflow (Step 3). Apply the Orbitrap mass-difference criterion Delta-M >= 1.66*M^1.5/(R*sqrt(M_R)) to compute the smallest resolvable mass difference at the monoisotopic mass of the compound. For each isotope (13C, 15N, 2H, 17O, 18O, 34S), enumerate its mass difference from the monoisotopic peak and set the per-isotope exclusion limit as the largest count whose cumulative mass difference stays below the smallest resolvable Delta-M; the original MDT criterion applies this per-isotope test only to the tracers 13C, 2H, and 15N. ElemCor's improvement replaces the independent per-isotope exclusion with a combined criterion: enumerate isotope combinations and sum their mass differences over the whole combination, so that a combination such as 2x17O + 1x18O whose total mass difference falls within the resolvable window is included in the correction matrix even though each isotope alone is near the limit. Write the per-isotope limits, smallest resolvable Delta-M, and per-combination inclusion decisions to a CSV for verification against a reference compound measured at a known resolution.

## Related tools

- **ElemCor** (Software that implements the resolution-dependent, combination-summing correction matrix construction based on MDT) — github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory providing the resolvable-Delta-M criterion that drives isotope exclusion/inclusion) — github.com/4dsoftware/elemcor
- **IsoCor** (Baseline correction method that does not consider resolution effects; used for comparison)

## Examples

```
from elemcor import correction_matrix; mat = correction_matrix(formula='C5H10N2O3', resolution=100000, analyzer='orbitrap', tracer='15N')
```

## Evaluation signals

- For glutamine (C5H10N2O3) at nominal resolution 100,000, the smallest resolvable Delta-M should be ~2.07e-3 Da.
- Per-isotope exclusion limits at that resolution should match the reference: 2 for 17O and 0 for 15N, 2H, and 18O.
- The combination 2x17O + 1x18O must be included in the correction matrix (as in Fig. 5b) because its summed mass difference stays within the resolvable window.
- Amount of correction must be monotonically dependent on instrument resolution: identical to IsoCor at extremely low resolution, no correction at extremely high resolution.
- Writing per-isotope limits and per-combination decisions to mdt_correction_limits.csv and asserting the glutamine reference values.

## Limitations

- The original MDT isotope-exclusion criterion is limited to the tracers 13C, 2H, and 15N; ElemCor's combined sum-of-mass-differences criterion is required to cover other isotopes (17O, 18O, 34S).
- Requires the nominal instrument resolution to be specified (Step 3); without it the resolvable Delta-M cannot be computed.
- When the unlabeled-sample step is skipped, ElemCor runs on mass difference theory only, so ULS-based deconvolution of natural abundance is not available.
- Validation was limited to a single organism (S. cerevisiae) and single tracer (15N ammonium) in experiments.

## Evidence

- [abstract] resolution-dependent MDT: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [intro] high resolution justifies exclusions: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
- [intro] ElemCor improves the two methods: "Here we present ElemCor with correction and improvement of those two methods and a user-friendly graphical interface."
- [methods] resolution input in workflow: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [methods] tracer elements supported: "In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [discussion] ElemCor vs IsoCor comparison: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [readme] monotonic dependence on resolution: "the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [readme] error when tracer absent: "the algorithm will report error if the tracer element is not present in the compound."
- [readme] MDT-only mode: "Step 2 is optional, and when it is not performed, ElemCor runs based on mass difference theory only."
- [readme] ElemCor reference: "ElemCor: accurate data analysis and enrichment calculation for high-resolution LC-MS stable isotope labeling experiments"
