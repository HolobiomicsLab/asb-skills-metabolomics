---
name: isotopologue-mass-difference-calculation
description: Use when building a resolution-dependent isotopologue correction matrix
  for a compound formula at a given nominal instrument resolution, and you must decide
  which same-nominal-mass isotopologs (e.g. 17O vs 15N/13C contributions) can be resolved
  and which correction-matrix columns to zero.
when_to_use_negative:
- The instrument resolution is extremely low — ElemCor becomes identical to IsoCor's
  combinatorial correction with no resolution effect, so MDT zeroing has no effect
- The tracer element is not present in the compound formula — the README notes the
  algorithm reports an error if the tracer element is absent (e.g. loading 15N data
  but selecting C or S as tracer)
- An unlabeled-sample-based (ULS/FluxFix-style) correction is required instead — that
  path deconvolves unlabeled-sample FAM rather than computing mass-difference limits
tools:
- name: ElemCor
  role: Implements the MDT resolution-dependent correction matrix construction and
    the improved sum-of-mass-differences criterion; MATLAB function adaptable to flux-analysis
    pipelines
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory supplying the Orbitrap Delta-M resolution criterion
    used to decide isotope exclusion from the correction matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:ed78dcdb57084ec388bced07f52e88451f583c9fc673ef1f1025a9a1d541ba5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MDT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# isotopologue-mass-difference-calculation

## Summary

Reconstructs the mass difference theory (MDT) resolution-dependent zeroing logic of the ElemCor correction matrix: computing the smallest resolvable mass difference (Delta-M) for the analyzer, deriving per-isotope exclusion limits, and applying ElemCor's improved sum-of-mass-differences criterion over isotope combinations.

## When to use

Use when building a resolution-dependent isotopologue correction matrix for a compound formula at a given nominal instrument resolution, and you must decide which same-nominal-mass isotopologs (e.g. 17O vs 15N/13C contributions) can be resolved and which correction-matrix columns to zero.

## When NOT to use

- The instrument resolution is extremely low — ElemCor becomes identical to IsoCor's combinatorial correction with no resolution effect, so MDT zeroing has no effect
- The tracer element is not present in the compound formula — the README notes the algorithm reports an error if the tracer element is absent (e.g. loading 15N data but selecting C or S as tracer)
- An unlabeled-sample-based (ULS/FluxFix-style) correction is required instead — that path deconvolves unlabeled-sample FAM rather than computing mass-difference limits

## Inputs

- Compound elemental formula (e.g. C5H10N2O3 for glutamine)
- Nominal instrument resolution (e.g. 100,000) and mass analyzer type (Step 3/Step 5 inputs of ElemCor)
- Tracer element selection (13C, 2H, 15N, 18O, or 34S per ElemCor Step 4)
- Per-isotope exact mass differences from the monoisotopic peak (13C, 15N, 2H, 17O, 18O)

## Outputs

- Smallest resolvable Delta-M at the monoisotopic mass (e.g. ~2.07e-3 Da for glutamine at R=100,000)
- Per-isotope exclusion limits (glutamine reference: 17O=2; 15N, 2H, 18O=0)
- Per-combination inclusion decisions for the correction matrix (e.g. 2x17O + 1x18O included as in Fig. 5b)
- mdt_correction_limits.csv containing per-isotope limits, smallest resolvable Delta-M, and per-combination inclusion decisions

## How to apply

Load the compound formula (e.g. glutamine, C5H10N2O3) and the nominal instrument resolution (e.g. 100,000, Orbitrap) as specified in ElemCor Step 3, and the mass analyzer as in Step 5. Implement the Orbitrap mass-difference criterion Delta-M >= 1.66*M^1.5/(R*sqrt(M_R)) and evaluate it at the monoisotopic mass to obtain the smallest resolvable Delta-M (expected ~2.07e-3 Da for glutamine at R=100,000). For each isotope relevant to the formula (13C, 15N, 2H, 17O, 18O), compute its mass difference from the monoisotopic peak and set the per-isotope exclusion limit to the largest count whose cumulative mass difference stays below the smallest resolvable Delta-M (glutamine reference: limit 2 for 17O, 0 for 15N, 2H, and 18O). Then apply ElemCor's improvement over original MDT: enumerate isotope combinations and sum their mass differences over the combination rather than excluding each isotope independently — e.g. the 2x17O + 1x18O combination whose cumulative Delta-M falls within the resolvable window is retained in the correction matrix, as shown in Fig. 5b. Rationale: modern high-resolving-power instruments can resolve same-nominal-mass isotopologs, so including all isotopologs (as IsoCor does) is no longer justified; correction strength should be monotonically dependent on resolution. ElemCor additionally adds a small mass-difference correction delta that balances all isotopes, whereas the original MDT isotope-exclusion criterion is limited to tracers 13C, 2H, and 15N.

## Related tools

- **ElemCor** (Implements the MDT resolution-dependent correction matrix construction and the improved sum-of-mass-differences criterion; MATLAB function adaptable to flux-analysis pipelines) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory supplying the Orbitrap Delta-M resolution criterion used to decide isotope exclusion from the correction matrix)

## Examples

```
python mdt_correction_limits.py --formula C5H10N2O3 --resolution 100000 --analyzer Orbitrap --out mdt_correction_limits.csv
```

## Evaluation signals

- At R=100,000 for glutamine (C5H10N2O3), the smallest resolvable Delta-M computed from the Orbitrap criterion equals ~2.07e-3 Da
- Per-isotope exclusion limits reproduce the glutamine reference: 2 for 17O and 0 for 15N, 2H, and 18O
- The 2x17O + 1x18O combination is retained (included in the correction matrix) under the sum-of-mass-differences criterion, matching Fig. 5b
- Correction strength is monotonically dependent on resolution: at extremely high resolution no correction is applied, at extremely low resolution the result matches IsoCor
- mdt_correction_limits.csv contains all three record types (per-isotope limits, smallest resolvable Delta-M, per-combination decisions) and matches the reference assertions

## Limitations

- The original MDT isotope-exclusion criterion is limited to the tracers 13C, 2H, and 15N; ElemCor's improvement with a balancing mass-difference delta extends coverage to all isotopes
- Excluding each isotope independently (original MDT) can wrongly exclude valid combinations such as 2x17O + 1x18O; the combination-level summation is required
- MDT is recommended for low-mass compounds for cost efficiency; ULS is recommended for high-mass compounds with relatively large spectral inaccuracy
- ElemCor was developed under MATLAB (2016b); running the compiled tool requires MATLAB Compiler Runtime, though MATLAB itself is not needed to run ElemCor.exe

## Evidence

- [intro] modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified.: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
- [methods] In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified.: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [methods] In Step 4, the tracer element is selected. In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction.: "In Step 4, the tracer element is selected. In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [abstract] The tool is based on mass difference theory (MDT) and information from unlabeled samples (ULS) to account for resolution effects.: "The tool is based on mass difference theory (MDT) and information from unlabeled samples (ULS) to account for resolution effects."
- [readme] IsoCor performs correction based on combinatorics without considering ressolution effect.: "IsoCor performs correction based on combinatorics without considering ressolution effect."
