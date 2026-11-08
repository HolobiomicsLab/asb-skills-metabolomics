---
name: nmr-pulse-sequence-signal-modeling
description: Use when building or validating forward signal models for 1D/2D NMR relaxation
  or diffusion experiments (T1/T2 relaxation, PFG diffusion) prior to multi-exponential
  fitting or regularized numerical inversion.
when_to_use_negative:
- The target signal requires harmonic inversion or non-exponential decay components,
  which NMRInversions.jl does not implement (harminv handles these instead)
- The data are already fully processed relaxation/diffusion distributions and the
  task is peak extraction or visualization, not kernel construction
- The pulse sequence deviates from the standard IR/SR/CPMG/PGSE closed forms (e.g.
  stimulated echo with finite gradient pulses)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Julia
  role: Language in which the exponential kernel functions are written and evaluated
    over logarithmically-spaced test grids
- name: NMRInversions
  role: Package whose invert() numerical inversion and expfit() multi-exponential
    fitting routines consume the kernel models implemented here
  repo: github.com/aris-mav/NMRInversions.jl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.07745
    title: NMRInversions.jl, a Julia package for time-domain Nuclear Magnetic Resonance
  source_content_hash: sha256:166e1105b293495365a3404ade8c873035daf9155c717f069b6f5b21224cfde9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - NMRInversions
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# nmr-pulse-sequence-signal-modeling

## Summary

Implement closed-form exponential signal-decay kernels for relaxation (IR, SR, CPMG) and diffusion (PGSE, FC) NMR pulse sequences as checkable Julia functions, mirroring the kernel matrices underlying NMRInversions.jl's numerical inversion routines.

## When to use

Use when building or validating forward signal models for 1D/2D NMR relaxation or diffusion experiments (T1/T2 relaxation, PFG diffusion) prior to multi-exponential fitting or regularized numerical inversion.

## When NOT to use

- The target signal requires harmonic inversion or non-exponential decay components, which NMRInversions.jl does not implement (harminv handles these instead)
- The data are already fully processed relaxation/diffusion distributions and the task is peak extraction or visualization, not kernel construction
- The pulse sequence deviates from the standard IR/SR/CPMG/PGSE closed forms (e.g. stimulated echo with finite gradient pulses)

## Inputs

- Grid of recovery/delay times t and relaxation times T1, T2 spanning orders of magnitude
- Grid of gradient b-values and diffusion coefficients D for PGSE
- Closed-form equations from the paper's theory section (BPP exponential decay forms)

## Outputs

- Julia kernel functions for IR, SR, CPMG, PGSE (and FC) in kernel_models.jl
- Maximum absolute deviation table verifying each model reproduces the closed-form expression across the full test grid

## How to apply

Write a Julia function for each pulse sequence per the tabulated equations: inversion recovery M = 1 - 2*exp(-t/T1), saturation recovery M = 1 - exp(-t/T1), CPMG M = exp(-t/T2), and PGSE M = exp(-b*D). Evaluate each function on a logarithmically-spaced grid of test values (t, T1, T2, b, D) spanning several orders of magnitude, matching the logarithmically-spaced relaxation/diffusion axes the package uses for kernel construction. Compare every output against a direct evaluation of the closed-form expression and record the maximum absolute deviation per model; validation requires deviation of 0 (exact to floating-point tolerance). Save the functions and deviations in kernel_models.jl, from which the kernel matrix fed to invert() and expfit() can be assembled.

## Related tools

- **Julia** (Language in which the exponential kernel functions are written and evaluated over logarithmically-spaced test grids)
- **NMRInversions** (Package whose invert() numerical inversion and expfit() multi-exponential fitting routines consume the kernel models implemented here) — github.com/aris-mav/NMRInversions.jl

## Examples

```
K_ir(t, T1) = 1 .- 2 .* exp.(.-t ./ T1');  dev = maximum(abs.(K_ir(t, T1) .- (1 .- 2 .* exp.(.-t ./ T1))))
```

## Evaluation signals

- Each implemented model reproduces the closed-form expression with maximum absolute deviation of 0 across the full test grid, up to floating-point tolerance
- Test grids for t, T1, T2, b, D span several orders of magnitude with logarithmic spacing, mirroring the package's kernel axes
- IR kernel approaches M = -1 at long t and M = 1 at t = 0; CPMG/PGSE kernels monotonically decay from 1 toward 0
- Kernel functions assemble into matrices whose row/column axes match the logarithmically-spaced relaxation/diffusion grids used by invert()

## Limitations

- NMRInversions.jl does not implement harmonic inversions; decay signals outside the exponential/BPP family must be handled by other packages such as harminv
- Closed-form kernels are exact only for ideal pulse sequences; experimental imperfections (imperfect 180° pulses, finite diffusion encoding) are not captured

## Evidence

- [other] kernel equations from theory: [evidence span withheld — anchor status: not_found]
- [other] log-spaced grid: [evidence span withheld — anchor status: not_found]
- [intro] inversion functionality: "This package can be used to easily perform numerical inversions for 1D and 2D NMR relaxation and diffusion measurements."
- [methods] main inversion function: "The most important function is `invert()`, which is the main function of the package"
- [abstract] harmonic inversion gap: "These can be handled by other packages such as "harminv" (Johnson, 2023), and are not yet implemented in NMRInversions.jl"
- [readme] exponential decay model: "This package can be used to easily perform numerical inversions for 1D and 2D NMR relaxation and diffusion measurements."
