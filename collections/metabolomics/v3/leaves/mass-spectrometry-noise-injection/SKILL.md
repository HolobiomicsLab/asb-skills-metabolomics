---
name: mass-spectrometry-noise-injection
description: Use when generating a synthetic mzML with SMITER and the simulated run
  must carry a realistic baseline intensity noise, e.g. to compare simulated vs. real
  TIC noise profiles as in the 24-nucleoside-standard validation.
when_to_use_negative:
- The target is a real measured LC-MS/MS dataset rather than a SMITER-simulated synthetic
  mzML — noise injection only applies to synthetic runs
- Only MS1 isotope-pattern accuracy is needed without baseline noise realism, and
  the added noise would complicate downstream validation of ground-truth peak intensities
- The molecule class has no SMITER fragmentation model and the goal is MS2-level realism,
  since noise injection alone will not supply missing fragmentation behavior
tools:
- name: SMITER
  role: Python library whose modular noise injector (smiter.noise_functions.UniformNoiseInjector)
    adds m/z- and intensity noise to the simulated run before writing the mzML
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Language in which the noise injector is configured and passed to smiter.synthetic_mzml.write_mzml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:efcaa0b63f19c4149ffe8ec8c47aa4f42e8166b544bda9439451cd7817ae5894
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-noise-injection

## Summary

Inject intensity noise into a simulated LC-MS/MS run using SMITER's modular noise injector (e.g. UniformNoiseInjector with ~1000 counts of white noise) so that the synthetic mzML carries a baseline noise profile that can be compared against real instrument measurements.

## When to use

Use when generating a synthetic mzML with SMITER and the simulated run must carry a realistic baseline intensity noise, e.g. to compare simulated vs. real TIC noise profiles as in the 24-nucleoside-standard validation.

## When NOT to use

- The target is a real measured LC-MS/MS dataset rather than a SMITER-simulated synthetic mzML — noise injection only applies to synthetic runs
- Only MS1 isotope-pattern accuracy is needed without baseline noise realism, and the added noise would complicate downstream validation of ground-truth peak intensities
- The molecule class has no SMITER fragmentation model and the goal is MS2-level realism, since noise injection alone will not supply missing fragmentation behavior

## Inputs

- Noise injector instance, e.g. smiter.noise_functions.UniformNoiseInjector configured with ~1000 counts of white noise
- Peak properties dict (modified nucleoside standards with chemical formulas and Modomics retention times) from smiter.lib.csv_to_peak_properties
- Fragmentor (e.g. a nucleoside fragmentation model with 10% fragment omission)
- General simulation parameters: gradient_length 80 min, scan time difference 60 ms, m=0.001701, y=0.2

## Outputs

- Simulated mzML file containing MS1 and MS2 peaks with injected baseline intensity noise
- Extracted TIC whose noise profile can be compared to the published real 24-standard measurement (Figure 3-style comparison)

## How to apply

Choose a noise generator as one of the SMITER simulation inputs, typically smiter.noise_functions.UniformNoiseInjector, configured with white noise on the order of 1000 counts. Pass the noise injector together with the peak properties dict (built via smiter.lib.csv_to_peak_properties from the example_data csv), a fragmentor (e.g. a nucleoside fragmentation model with ~10% fragment omission), and general parameters (gradient_length 80 min, scan time difference 60 ms, m=0.001701, y=0.2) into smiter.synthetic_mzml.write_mzml. Because SMITER's design is modular, the default uniform white noise can be swapped for or extended by other intensity-specific noise models without changing the rest of the pipeline. After writing the mzML, extract the TIC and verify the noise level against the real measurement — in the published 24-nucleoside-standard comparison, the simulated run correctly contains only the injected white noise but exhibits lower noise than the real run, which additionally shows an unidentified ~15 min contaminant peak.

## Related tools

- **SMITER** (Python library whose modular noise injector (smiter.noise_functions.UniformNoiseInjector) adds m/z- and intensity noise to the simulated run before writing the mzML) — https://github.com/LeidelLab/SMITER
- **Python** (Language in which the noise injector is configured and passed to smiter.synthetic_mzml.write_mzml)

## Examples

```
from smiter.noise_functions import UniformNoiseInjector; noise_injector = UniformNoiseInjector(white_noise=1000); smiter.synthetic_mzml.write_mzml(peak_properties, fragmentor, noise_injector, gradient_length=80*60)
```

## Evaluation signals

- The simulated mzML's TIC contains the basic injected white noise (e.g. the configured ~1000 counts uniform background)
- TIC noise profile of the simulated run is visibly lower than that of the published real 24-nucleoside-standard measurement, matching the reported observation
- The simulated run lacks real-measurement artifacts such as the unidentified ~15 min contaminant peak
- mzML is produced end-to-end via smiter.synthetic_mzml.write_mzml with peak_properties, fragmentor, and noise injector all passed as parameters
- Changing the noise injector (or its counts parameter) changes the baseline intensity noise in the output mzML, confirming the modular noise step took effect

## Limitations

- With the default uniform white noise (~1000 counts), the simulated run exhibits lower noise than the corresponding real measurement, which contains additional unidentified noise and a ~15 min contaminant peak
- The discussion-section prose is absent from the extracted article text, so no author-stated limitations on the noise models are available
- Realistic intensity-specific noise beyond the uniform injector requires implementing a custom noise model, though SMITER's modular design is stated to make this easy

## Evidence

- [readme] m/z-and intensity noise injection ( uniform noise or a noise model that combines general noise with intensity-specific noise): [evidence span withheld — anchor status: external_doc]
- [methods] Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector): "Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [abstract] SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models.: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [methods] Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
