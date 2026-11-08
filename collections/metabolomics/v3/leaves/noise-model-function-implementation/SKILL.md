---
name: noise-model-function-implementation
description: Use when implementing, reproducing, or validating SMITER's intensity-specific
  chromatographic peak-noise model — i.e., when you need sigma as a function of signal
  intensity for synthetic mzML generation or noise-model comparison.
when_to_use_negative:
- You need uniform, intensity-independent noise — use the default smiter.noise_functions.UniformNoiseInjector
  instead of the intensity-specific model.
- You only need to run an end-to-end SMITER simulation with existing noise generators
  rather than implement or validate a new noise model.
- Your noise model depends on m/z rather than signal intensity — SMITER separates
  m/z noise injection from intensity-specific noise.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: SMITER
  role: Library whose noise_functions injector interface (e.g. UniformNoiseInjector)
    the custom noise models must mirror for drop-in use with smiter.synthetic_mzml.write_mzml
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Implementation language for the noise-function module and the pytest test
    suite
- name: pytest
  role: Test framework used to assert bounds (sigma(1) = 0.001701) and monotonic decrease
    of the power-law noise model
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:7f85b18d434ca69a059da0ad92eef349fa462c00163b7a469dd7d7dffd7df0a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  - pytest
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# noise-model-function-implementation

## Summary

Reconstruct SMITER's intensity-dependent peak-noise model by implementing the saturating sigma = m(1 - e^{-ci}) + d noise function (Eq. 1-2) as a standalone Python module with pytest-verified bounds and monotonicity, mirroring the smiter.noise_functions injector interface.

## When to use

Use when implementing, reproducing, or validating SMITER's intensity-specific chromatographic peak-noise model — i.e., when you need sigma as a function of signal intensity for synthetic mzML generation or noise-model comparison.

## When NOT to use

- You need uniform, intensity-independent noise — use the default smiter.noise_functions.UniformNoiseInjector instead of the intensity-specific model.
- You only need to run an end-to-end SMITER simulation with existing noise generators rather than implement or validate a new noise model.
- Your noise model depends on m/z rather than signal intensity — SMITER separates m/z noise injection from intensity-specific noise.

## Inputs

- Noise-model parameters: m, c, d for the saturating model sigma = m(1-e^{-ci}) + d
- Noise-model parameters: m, y for the Mspire-style power-law model sigma = m*i^{-y} (published nucleoside values m = 0.001701, y = 0.2)
- Signal intensities i of molecules in the simulated LC-MS/MS run
- Peak properties dict (optionally created via smiter.lib.csv_to_peak_properties) when wiring the injector into a full simulation

## Outputs

- Python noise-function module compatible with smiter.noise_functions injector interface
- pytest unit-test suite asserting bounds and monotonicity of sigma
- Intensity-dependent sigma values (noise standard deviations) usable as input to synthetic mzML writing

## How to apply

Create a standalone Python module exposing the saturating noise function sigma = m(1 - e^{-ci}) + d, where m and d are noise-injection parameters and c controls how sigma increases with intensity; expose m, c, and d as callable parameters. In the same module, add the Mspire-style power-law intensity-specific noise function sigma = m*i^{-y}, mirroring the interface of the noise injectors in smiter.noise_functions such as UniformNoiseInjector, so either model can be swapped in as a noise generator for smiter.synthetic_mzml.write_mzml. Write pytest unit tests instantiating the power-law model with the published nucleoside parameters m = 0.001701 and y = 0.2, asserting sigma(1) = 0.001701 via pytest.approx, sigma approaching 0 as intensity grows, and monotonic decrease over a logarithmically spaced intensity range. This modularity is deliberate: SMITER's design allows noise and fragmentation models to be easily implemented or adapted, so custom noise injectors should follow the existing injector interface to remain drop-in compatible.

## Related tools

- **SMITER** (Library whose noise_functions injector interface (e.g. UniformNoiseInjector) the custom noise models must mirror for drop-in use with smiter.synthetic_mzml.write_mzml) — https://github.com/LeidelLab/SMITER
- **Python** (Implementation language for the noise-function module and the pytest test suite)
- **pytest** (Test framework used to assert bounds (sigma(1) = 0.001701) and monotonic decrease of the power-law noise model)

## Examples

```
from smiter.noise_functions import UniformNoiseInjector; sigma = lambda i, m=0.001701, y=0.2: m * i**(-y); assert sigma(1) == pytest.approx(0.001701)
```

## Evaluation signals

- Power-law model returns sigma(1) = m = 0.001701 at unit intensity (pytest.approx comparison)
- sigma = m*i^{-y} decreases monotonically over a logarithmically spaced intensity range
- sigma approaches 0 as intensity grows for the power-law model
- Saturating model sigma = m(1-e^{-ci}) + d yields minimum sigma = d and maximum sigma = m + d at extreme intensities
- pytest suite reports all tests passing

## Limitations

- Noise models are intensity-specific only; SMITER separates this from m/z noise injection (uniform noise or combined general-plus-intensity-specific noise).
- Parameter values such as m = 0.001701 and y = 0.2 are nucleoside-specific published fits; other molecule classes may need re-fitting.
- The published article provides no changelog/discussion prose covering limitations of the noise model.

## Evidence

- [results] SMITER's intensity-specific noise model (equation 1, sigma = m(1 - e^(-ci)) + d) defines sigma via: i, the intensity of the molecule: [evidence span withheld — anchor status: not_found]
- [methods] Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector): "3. Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [readme] m/z-and intensity noise injection ( uniform noise or a noise model that combines general noise with intensity-specific noise): [evidence span withheld — anchor status: external_doc]
- [abstract] SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models: "As SMITER features a modular design, noise and fragmentation models can easily be implemented or adapted."
