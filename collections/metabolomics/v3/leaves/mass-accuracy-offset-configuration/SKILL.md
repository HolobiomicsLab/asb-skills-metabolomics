---
name: mass-accuracy-offset-configuration
description: 'Use when simulating LC-MS/MS data with SMITER and you need to model
  instrument mass-accuracy error: e.g., benchmarking quantification robustness of
  isomeric nucleosides (Cm, m3C, m5C) across ppm offsets from 0 to 4.'
when_to_use_negative:
- When the goal is to model chromatographic interference such as co-elution or co-fragmentation
  rather than mass-accuracy error — vary other simulation parameters instead
- When analyzing real experimental LC-MS/MS data, since this skill only applies to
  SMITER-simulated synthetic mzML generation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMITER
  role: Python library used to simulate LC-MS/MS runs with a configurable mass-accuracy
    offset (MACHINE_OFFSET_IN_PPM) and write synthetic mzML output
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Runtime environment (Python 3.7+) in which the SMITER simulation scripts are
    executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:f76108f257d9dfb65b846bed64bf63847dd2794517c64c3f163c79dcb2207015
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-accuracy-offset-configuration

## Summary

Configure SMITER's machine mass-accuracy offset (MACHINE_OFFSET_IN_PPM) to inject controlled ppm errors into simulated LC-MS/MS runs, enabling paired 0-ppm vs 4-ppm gold-standard mzML comparisons that quantify how mass measurement imprecision degrades isotope-pattern-based quantification.

## When to use

Use when simulating LC-MS/MS data with SMITER and you need to model instrument mass-accuracy error: e.g., benchmarking quantification robustness of isomeric nucleosides (Cm, m3C, m5C) across ppm offsets from 0 to 4.

## When NOT to use

- When the goal is to model chromatographic interference such as co-elution or co-fragmentation rather than mass-accuracy error — vary other simulation parameters instead
- When analyzing real experimental LC-MS/MS data, since this skill only applies to SMITER-simulated synthetic mzML generation

## Inputs

- example_data CSV with peak properties of modified nucleosides (Cm, m3C, m5C)
- SMITER simulation parameters (gradient_length, fragmentor, noise generator)
- MACHINE_OFFSET_IN_PPM values (0 ppm and 4 ppm)

## Outputs

- Two synthetic mzML files: one simulated at 0 ppm offset, one at 4 ppm offset
- Isotope-pattern match scores per spectrum for each run, showing degraded quantification quality at 4 ppm

## How to apply

First build the peak_properties dict from the example_data CSV via smiter.lib.csv_to_peak_properties for the modified nucleosides Cm, m3C, and m5C. Choose a nucleoside fragmentor, a smiter.noise_functions.UniformNoiseInjector noise generator, and set general simulation parameters such as gradient_length. Set MACHINE_OFFSET_IN_PPM to 0 ppm and run the simulation, writing the result with smiter.synthetic_mzml.write_mzml to obtain the baseline run; then repeat the identical simulation with MACHINE_OFFSET_IN_PPM set to 4 ppm. The rationale is that a controlled, isolated ppm offset lets you attribute quantification degradation solely to mass-accuracy error rather than other noise sources. Validate by scoring isotope-pattern matches on both runs: the 0-ppm run should yield optimal quantification, while the 4-ppm run should show a high number of spectra with low isotope-pattern match scores, reproducing Figure 5A/B.

## Related tools

- **SMITER** (Python library used to simulate LC-MS/MS runs with a configurable mass-accuracy offset (MACHINE_OFFSET_IN_PPM) and write synthetic mzML output) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime environment (Python 3.7+) in which the SMITER simulation scripts are executed)

## Examples

```
from smiter.synthetic_mzml import write_mzml; write_mzml(peak_properties=props, noise_injector=UniformNoiseInjector(), fragmentor=NucleosideFragmentor(), gradient_length=..., MACHINE_OFFSET_IN_PPM=4)
```

## Evaluation signals

- Both generated mzML files (0 ppm and 4 ppm) must be loadable/parseable as valid mzML
- The 0-ppm run yields optimal quantification, while the 4-ppm run shows a high number of spectra with low isotope-pattern match scores
- Scored isotope-pattern matches on the two runs reproduce the degradation pattern of Figure 5A/B
- The two simulations are identical except for MACHINE_OFFSET_IN_PPM, so any quantification difference is attributable to the ppm offset

## Limitations

- Quantification quality is only optimal at zero ppm offset; any non-zero offset introduces many low isotope-pattern match score spectra
- Only the 0–4 ppm range is demonstrated; sensitivity beyond 4 ppm or for other molecule classes must be re-evaluated

## Evidence

- [results] Increasing measurement imprecision degrades quantification quality: without ppm errors quantifications are optimal, while a non-zero ppm offset produces a high number of spectra with low isotope-pattern match scores.: [evidence span withheld — anchor status: not_found]
- [methods] Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [abstract] It enables the simulation of any biomolecule amenable to mass spectrometry (MS) since all calculations are based on chemical formulas.: "It enables the simulation of any biomolecule amenable to mass spectrometry (MS) since all calculations are based on chemical formulas."
