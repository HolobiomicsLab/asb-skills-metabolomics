---
name: exponential-decay-function-implementation
description: Use when building or validating kernel functions for relaxation (T1,
  T2) or diffusion (D) NMR signals before feeding them to regularized numerical inversion
  (invert()) or multi-exponential fitting (expfit).
when_to_use_negative:
- Signals requiring harmonic inversion (damped oscillations) — these are not implemented
  in NMRInversions.jl and must be handled by packages such as harminv
- Fitting real experimental multi-exponential data — use the package's invert() or
  expfit routines instead of hand-coded kernels
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: Julia
  role: Language in which the exponential decay kernel functions are written and verified
    over test grids
  repo: https://Julialang.org/downloads/
- name: NMRInversions
  role: Package whose numerical inversion and kernel axes the implemented models must
    be consistent with; installed via Pkg.add("NMRInversions")
  repo: https://github.com/aris-mav/NMRInversions.jl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.07745
    title: NMRInversions.jl, a Julia package for time-domain Nuclear Magnetic Resonance
  source_content_hash: sha256:3adf9897ebc0c7c4e24229554ee99537e3b54faac79cde0375a4e27ef857c86b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - NMRInversions
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# exponential-decay-function-implementation

## Summary

Implements the closed-form exponential decay kernels for NMR relaxation and diffusion pulse sequences (inversion recovery, saturation recovery, CPMG, PGSE) as checkable Julia functions, mirroring the kernel model forms that NMRInversions.jl's numerical inversion and multi-exponential fitting routines rely on.

## When to use

Use when building or validating kernel functions for relaxation (T1, T2) or diffusion (D) NMR signals before feeding them to regularized numerical inversion (invert()) or multi-exponential fitting (expfit).

## When NOT to use

- Signals requiring harmonic inversion (damped oscillations) — these are not implemented in NMRInversions.jl and must be handled by packages such as harminv
- Fitting real experimental multi-exponential data — use the package's invert() or expfit routines instead of hand-coded kernels

## Inputs

- Grids of test values for time t, relaxation times T1 and T2, PGSE b-factor, and diffusion coefficient D (logarithmically spaced over orders of magnitude)
- Closed-form model equations from the paper's theory section

## Outputs

- Julia functions for IR, SR, CPMG, and PGSE decay models in kernel_models.jl
- Per-model maximum absolute deviation values versus direct closed-form evaluation

## How to apply

Write a Julia function for each pulse-sequence model: inversion recovery M = 1 - 2*exp(-t/T1), saturation recovery M = 1 - exp(-t/T1), CPMG M = exp(-t/T2), and PGSE M = exp(-b*D), following the equations tabulated in the theory section. Evaluate each function over a logarithmically-spaced grid of test values (t, T1, T2, b, D) spanning several orders of magnitude, mirroring the logarithmically-spaced relaxation/diffusion axes the package uses for kernel construction. Compare every output against a direct evaluation of the closed-form expression and record the maximum absolute deviation per model. Save the functions and verification deviations in kernel_models.jl; validation requires a maximum absolute deviation of 0 (exact match up to floating-point tolerance) across the full test grid.

## Related tools

- **Julia** (Language in which the exponential decay kernel functions are written and verified over test grids) — https://Julialang.org/downloads/
- **NMRInversions** (Package whose numerical inversion and kernel axes the implemented models must be consistent with; installed via Pkg.add("NMRInversions")) — https://github.com/aris-mav/NMRInversions.jl

## Examples

```
M_ir(t, T1) = 1 - 2*exp(-t ./ T1); M_cpmg(t, T2) = exp(-t ./ T2); M_pgse(b, D) = exp(-b .* D); maximum(abs.(M_ir.(t, T1) .- (1 .- 2 .* exp.(-t ./ T1))))
```

## Evaluation signals

- Maximum absolute deviation between each implemented function and a direct evaluation of the closed-form expression is 0 across the full test grid (up to floating-point tolerance)
- All four pulse-sequence models (IR, SR, CPMG, PGSE) are covered by the test grid
- Test grids span several orders of magnitude in t, T1, T2, b, and D, matching logarithmically-spaced relaxation/diffusion axes
- Functions and verification deviations are persisted in kernel_models.jl

## Limitations

- Per BPP theory the kernels capture exponentially-decaying signals; non-exponential behavior (e.g., damped oscillations needing harmonic inversion) is out of scope
- Harmonic inversions are not implemented in NMRInversions.jl at the time of writing

## Evidence

- [results] The paper describes relaxation and diffusion NMR experiments as yielding exponentially-decaying signals (per BPP theory), and states that multi-exponential forms are commonly used to characterize the data: [evidence span withheld — anchor status: not_found]
- [methods] invert() is the main function of the package: "The most important function is `invert()`, which is the main function of the package"
- [readme] This package can be used to easily perform numerical inversions for 1D and 2D NMR relaxation and diffusion measurements.: "This package can be used to easily perform numerical inversions for 1D and 2D NMR relaxation and diffusion measurements."
- [abstract] Harmonic inversions are handled by other packages, not NMRInversions.jl: "These can be handled by other packages such as "harminv" (Johnson, 2023), and are not yet implemented in NMRInversions.jl"
