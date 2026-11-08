---
name: smiter-noise-injector-interface-design
description: 'Use when extending SMITER with a custom noise model: an intensity-dependent
  sigma(i) must satisfy the sigma = m(1 - e^{-ci}) + d or sigma = m*i^{-y} form and
  be exposed via the smiter.noise_functions injector interface (e.g. UniformNoiseInjector)
  with unit tests.'
when_to_use_negative:
- The goal is to run a complete LC-MS/MS simulation and write an mzML file — use the
  full SMITER pipeline (csv_to_peak_properties, fragmentor, noise injector, smiter.synthetic_mzml.write_mzml),
  not a standalone noise-module rebuild
- The task is analyzing real experimental LC-MS/MS data (e.g. existing mzML peak lists)
  — this skill models/simulates noise for synthetic mzML generation, it does not estimate
  noise from measured spectra
- The default UniformNoiseInjector (uniform noise) already meets requirements — only
  build a new intensity-dependent injector when intensity-specific noise behavior
  is the target
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SMITER
  role: Target library whose modular smiter.noise_functions injector interface (e.g.
    UniformNoiseInjector) the new noise models must conform to
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Implementation language for the noise module and pytest unit tests
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:42808cee16390de6fe2fd2ababf2bf52842776a3a8ebec7bd4348f8cc8e3fe7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# smiter-noise-injector-interface-design

## Summary

Implement new intensity-dependent noise-injection modules for SMITER (sigma = m(1 - e^{-ci}) + d or the Mspire-style sigma = m*i^{-y}) behind the smiter.noise_functions injector interface, validated with pytest using the published nucleoside parameters (m = 0.001701, y = 0.2).

## When to use

Use when extending SMITER with a custom noise model: an intensity-dependent sigma(i) must satisfy the sigma = m(1 - e^{-ci}) + d or sigma = m*i^{-y} form and be exposed via the smiter.noise_functions injector interface (e.g. UniformNoiseInjector) with unit tests.

## When NOT to use

- The goal is to run a complete LC-MS/MS simulation and write an mzML file — use the full SMITER pipeline (csv_to_peak_properties, fragmentor, noise injector, smiter.synthetic_mzml.write_mzml), not a standalone noise-module rebuild
- The task is analyzing real experimental LC-MS/MS data (e.g. existing mzML peak lists) — this skill models/simulates noise for synthetic mzML generation, it does not estimate noise from measured spectra
- The default UniformNoiseInjector (uniform noise) already meets requirements — only build a new intensity-dependent injector when intensity-specific noise behavior is the target

## Inputs

- Noise-model equations and parameters (Eq. 1: sigma = m(1 - e^{-ci}) + d with m, c, d; Mspire-style: sigma = m*i^{-y} with m = 0.001701 and y = 0.2)
- Intensity values (scalar or arrays, including logarithmically spaced ranges) at which sigma(i) is evaluated
- SMITER noise injector interface conventions (smiter.noise_functions.UniformNoiseInjector)

## Outputs

- Standalone Python module exposing the two noise-injector functions/classes with m, c, d (and y) as parameters
- pytest unit-test module asserting bounds (sigma(1) = 0.001701, sigma -> 0), monotonicity over log-spaced intensities, and range [d, m + d]
- All-passing pytest test report

## How to apply

Create a standalone Python module that defines the additive/saturating noise function sigma = m(1 - e^{-ci}) + d, exposing m, c, and d as parameters, where c controls the increase of sigma with intensity, d is the minimum sigma, and m + d the maximum sigma. Add a second function mirroring the Mspire-style intensity-specific model sigma = m*i^{-y}, matching the interface of the existing injectors in smiter.noise_functions such as UniformNoiseInjector, so either class can be dropped into step 3 ('Choose a Noise generator') of the SMITER simulation workflow. Write pytest unit tests instantiating both models with the published nucleoside parameters m = 0.001701 and y = 0.2. Assert that sigma(1) approx equals m = 0.001701 (pytest.approx) and approaches 0 as intensity grows, that sigma is monotonically decreasing over a logarithmically spaced intensity range, and that the saturating model's sigma stays within [d, m + d]. Run the suite with pytest and report all-passing results; this mirrors SMITER's stated modular design that lets noise models be 'easily implemented or adapted'.

## Related tools

- **SMITER** (Target library whose modular smiter.noise_functions injector interface (e.g. UniformNoiseInjector) the new noise models must conform to) — https://github.com/LeidelLab/SMITER
- **Python** (Implementation language for the noise module and pytest unit tests)

## Examples

```
pip install smiter; pytest -v
```

## Evaluation signals

- pytest reports all tests passing after implementation
- Power-law model bound: sigma(i=1) == m == 0.001701 within pytest.approx tolerance, and sigma -> 0 as intensity grows
- sigma(i) is monotonically decreasing across a logarithmically spaced intensity range for sigma = m*i^{-y}
- Saturating model sigma = m(1 - e^{-ci}) + d stays bounded within [d, m + d] for all tested intensities
- New injector can be substituted for UniformNoiseInjector in the SMITER workflow step 'Choose a Noise generator' without interface mismatch

## Limitations

- Validated numeric anchors (m = 0.001701, y = 0.2) come from the nucleoside fragmentation/noise example, so they parameterize nucleoside-scale intensities rather than arbitrary molecule classes
- No changelog was found for the SMITER repository, so interface drift between versions of smiter.noise_functions may not be traceable
- Discussion-section prose (limitations from the authors) is absent in the available text, so caveats beyond noise-bounds behavior must be established empirically

## Evidence

- [abstract] SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models.: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [methods] Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector): "Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [readme] m/z-and intensity noise injection ( uniform noise or a noise model that combines general noise with intensity-specific noise): [evidence span withheld — anchor status: external_doc]
