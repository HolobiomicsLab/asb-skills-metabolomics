---
name: lc-ms-intensity-noise-modeling
description: Use when simulating synthetic LC-MS/MS peaks in mzML and noise standard
  deviation must scale with signal intensity (e.g. reproducing SMITER's noise model
  or validating a replacement injector).
when_to_use_negative:
- The target signal is uniform (intensity-independent) noise — SMITER's default UniformNoiseInjector
  already covers that case without an intensity-dependent model
- The analysis consumes real experimental LC-MS/MS data rather than generating synthetic
  mzML gold standards — this skill models simulated noise only
- Noise must be applied to m/z rather than intensity — SMITER treats m/z noise and
  intensity noise as separate injection concerns
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: SMITER
  role: Host library whose modular noise injector interface (smiter.noise_functions,
    e.g. UniformNoiseInjector) the reconstructed models must mirror; injectors feed
    smiter.synthetic_mzml.write_mzml
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Implementation and pytest testing language for the standalone noise-function
    module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:4fb047dec0f8aa62b85be5c57acf60ea5a19b612cd83f154520bdef7c0c65b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lc-ms-intensity-noise-modeling

## Summary

Reconstruct and unit-test LC-MS/MS intensity-dependent noise models from SMITER (Eqs. 1-2): a saturating additive model sigma = m(1 - e^{-ci}) + d and a Mspire-style power-law model sigma = m*i^{-y}, packaged as reusable injectors in the smiter.noise_functions interface.

## When to use

Use when simulating synthetic LC-MS/MS peaks in mzML and noise standard deviation must scale with signal intensity (e.g. reproducing SMITER's noise model or validating a replacement injector).

## When NOT to use

- The target signal is uniform (intensity-independent) noise — SMITER's default UniformNoiseInjector already covers that case without an intensity-dependent model
- The analysis consumes real experimental LC-MS/MS data rather than generating synthetic mzML gold standards — this skill models simulated noise only
- Noise must be applied to m/z rather than intensity — SMITER treats m/z noise and intensity noise as separate injection concerns

## Inputs

- Intensity values (i) of simulated chromatographic peaks
- Noise model parameters: m, c, d for the saturating model (Eq. 1); m, y for the power-law model (published nucleoside values m = 0.001701, y = 0.2)
- Peak properties dict, optionally produced via smiter.lib.csv_to_peak_properties from a csv in example_data

## Outputs

- Standalone Python noise-injection module with intensity-dependent sigma functions matching the smiter.noise_functions injector interface
- pytest unit-test suite asserting parameter bounds and monotonic intensity dependence
- Sigma values consumable by smiter.synthetic_mzml.write_mzml when generating synthetic mzML

## How to apply

Implement the saturating noise model sigma = m(1 - e^{-ci}) + d, where i is molecule intensity, c controls the increase of sigma with intensity, d is the minimum sigma, and m + d is the maximum sigma, exposing m, c and d as parameters. Add the Mspire-style power-law alternative sigma = m*i^{-y}, mirroring the interface of existing injectors such as smiter.noise_functions.UniformNoiseInjector, so either can be dropped into smiter.synthetic_mzml.write_mzml. Write pytest unit tests instantiating both models with the published nucleoside parameters m = 0.001701 and y = 0.2. Assert the bounds of the power-law model (sigma(1) = m at unit intensity, sigma approaching 0 as intensity grows, via pytest.approx) and assert monotonic decrease of sigma over a logarithmically spaced intensity range. Run the suite with pytest and report all-passing results before wiring the injector into a full mzML simulation.

## Related tools

- **SMITER** (Host library whose modular noise injector interface (smiter.noise_functions, e.g. UniformNoiseInjector) the reconstructed models must mirror; injectors feed smiter.synthetic_mzml.write_mzml) — https://github.com/LeidelLab/SMITER
- **Python** (Implementation and pytest testing language for the standalone noise-function module)

## Examples

```
pytest tests/test_noise_functions.py -k "noise_injector"; sigma = m * i ** (-y)  # m=0.001701, y=0.2
```

## Evaluation signals

- sigma(1) equals m = 0.001701 for the power-law model at unit intensity (pytest.approx comparison)
- sigma of the power-law model approaches 0 as intensity grows, checked with a tolerance-appropriate comparison
- sigma = m*i^{-y} decreases monotonically across a logarithmically spaced intensity range
- Saturating model sigma = m(1 - e^{-ci}) + d stays bounded between d (minimum) and m + d (maximum)
- New injector is callable through the same interface as smiter.noise_functions.UniformNoiseInjector and passes the pytest suite with all tests green

## Limitations

- SMITER's discussion section prose is absent from the indexed text, so published caveats on the noise models' realism could not be extracted
- Parameter values (m = 0.001701, y = 0.2) are published for nucleosides; transfer to other biomolecule classes is untested in this context
- SMITER requires Python 3.7 or higher (LeidelLab README) and recommends a virtual environment for installation

## Evidence

- [results] SMITER's intensity-specific noise model (equation 1, sigma = m(1 - e^(-ci)) + d) defines sigma via: i, the intensity of the molecule; c, which controls the increase in sigma with increasing intensity; d, the minimum sigma; and m + d, the maximum sigma.: [evidence span withheld — anchor status: not_found]
- [abstract] SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models.: "As SMITER features a modular design, noise and fragmentation models can easily be implemented or adapted."
- [methods] Choose a Noise generator: "Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [readme] m/z-and intensity noise injection ( uniform noise or a noise model that combines general noise with intensity-specific noise): [evidence span withheld — anchor status: external_doc]
- [methods] Run the simulation and write the resulting mzML: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml`"
