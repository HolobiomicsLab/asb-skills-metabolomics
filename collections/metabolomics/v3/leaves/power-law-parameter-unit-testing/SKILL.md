---
name: power-law-parameter-unit-testing
description: Use when reimplementing or refactoring SMITER noise injection (e.g. UniformNoiseInjector-style
  injectors) and you need to verify that an intensity-dependent noise sigma (power-law
  m*i^{-y} or saturating form) reproduces published parameters and behaves monotonically.
when_to_use_negative:
- The goal is to run an end-to-end SMITER LC-MS/MS simulation and write mzML output
  via smiter.synthetic_mzml.write_mzml rather than to validate noise-model parameters
  in isolation.
- The target noise model is intensity-independent (e.g. pure uniform noise injection),
  where sigma bounds and monotonic intensity dependence do not apply.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: SMITER
  role: Source library whose noise injector interface (smiter.noise_functions, e.g.
    UniformNoiseInjector) the standalone module mirrors
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Implementation and pytest testing language for the noise module and its unit
    tests
  repo: https://github.com/LeidelLab/SMITER
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:0c8fe93f096d107a1712fd880c1bacfee8f4509cd1dd7bb6421e82b979a4a1e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# power-law-parameter-unit-testing

## Summary

Implement SMITER's intensity-specific noise models as a standalone Python module and validate their parameters with pytest unit tests, asserting the bounds and monotonic behavior of sigma vs. intensity.

## When to use

Use when reimplementing or refactoring SMITER noise injection (e.g. UniformNoiseInjector-style injectors) and you need to verify that an intensity-dependent noise sigma (power-law m*i^{-y} or saturating form) reproduces published parameters and behaves monotonically.

## When NOT to use

- The goal is to run an end-to-end SMITER LC-MS/MS simulation and write mzML output via smiter.synthetic_mzml.write_mzml rather than to validate noise-model parameters in isolation.
- The target noise model is intensity-independent (e.g. pure uniform noise injection), where sigma bounds and monotonic intensity dependence do not apply.

## Inputs

- Published noise-model parameters (m = 0.001701, y = 0.2 from the SMITER nucleoside noise model)
- Parameters m, c, d for SMITER Eq. 1 sigma = m(1 - e^{-ci}) + d
- Logarithmically spaced intensity range (array of intensity values)

## Outputs

- Standalone Python noise module mirroring the smiter.noise_functions injector interface
- pytest test suite asserting sigma bounds and monotonic decrease
- Passing pytest report confirming parameter correctness

## How to apply

Create a standalone Python module exposing two additive noise functions with a common interface mirroring smiter.noise_functions (e.g. UniformNoiseInjector): (1) SMITER Eq. 1, sigma = m(1 - e^{-ci}) + d, where i is the molecule intensity, c controls the increase of sigma with intensity, d is the minimum sigma, and m + d is the maximum sigma; and (2) the Mspire-style power-law sigma = m * i^{-y}. Write pytest unit tests that instantiate both models with the published nucleoside parameters m = 0.001701 and y = 0.2. Assert the bounds of the power-law model using pytest.approx: sigma(1) must equal m = 0.001701 at unit intensity, and sigma must approach 0 as intensity grows. Then assert monotonic decrease of sigma over a logarithmically spaced intensity range to confirm the intensity dependence. Run the suite with pytest (or tox, as the SMITER README prescribes for package testing) and report all-passing results.

## Related tools

- **SMITER** (Source library whose noise injector interface (smiter.noise_functions, e.g. UniformNoiseInjector) the standalone module mirrors) — https://github.com/LeidelLab/SMITER
- **Python** (Implementation and pytest testing language for the noise module and its unit tests) — https://github.com/LeidelLab/SMITER

## Examples

```
pytest -q tests/test_noise_models.py
```

## Evaluation signals

- sigma(1) == 0.001701 within pytest.approx tolerance for sigma = m*i^{-y}
- sigma approaches 0 as intensity grows (power-law model), with tolerance-appropriate comparison
- sigma decreases monotonically across a logarithmically spaced intensity range
- For Eq. 1, sigma is bounded below by d and above by m + d across the intensity range
- Full pytest run reports all tests passing

## Limitations

- No changelog is available for SMITER, so version drift in the noise-function API cannot be ruled out.
- SMITER's modular design means noise and fragmentation models can be swapped; tests must be tied to a specific model's published parameters.
- Validated parameters (m = 0.001701, y = 0.2) are specific to the published nucleoside noise model and need not transfer to other molecule classes.

## Evidence

- [full_text] SMITER's intensity-specific noise model (equation 1, sigma = m(1 - e^(-ci)) + d): [evidence span withheld — anchor status: not_found]
- [full_text] parameters m and y definition: [evidence span withheld — anchor status: not_found]
- [methods] Choose a Noise generator: "3. Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [abstract] modular design: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [readme] README feature description of noise injection: [evidence span withheld — anchor status: external_doc]
- [readme] testing via tox: "To test the package and correct installation::

    user@localhost:~/smiter$ tox"
