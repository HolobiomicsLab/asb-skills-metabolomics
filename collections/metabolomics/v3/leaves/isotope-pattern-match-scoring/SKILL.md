---
name: isotope-pattern-match-scoring
description: Use when comparing synthetic or real LC-MS/MS runs of known molecules
  (e.g. nucleosides Cm, m3C, m5C) to assess how mass-accuracy errors (ppm offsets)
  affect isotope-pattern match scores and quantification quality.
when_to_use_negative:
- The input run was not produced by a formula-based simulator like SMITER and there
  is no known ground truth (chemical formulas) to compute theoretical isotope patterns
  against.
- The goal is fragment-ion (MS2) identity rather than precursor isotope-pattern matching
  — SMITER's peptide/nucleoside fragmentation models address that separately.
- Only a single run with no error-free (0 ppm) counterpart exists, so the effect of
  the ppm offset on quantification quality cannot be assessed comparatively.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMITER
  role: Generates the synthetic mzML LC-MS/MS runs (0 ppm and 4 ppm MACHINE_OFFSET_IN_PPM)
    whose isotope-pattern matches are scored; isotope patterns are computed with pyQms.
  repo: https://github.com/LeidelLab/SMITER
- name: pyQms
  role: Provides the highly-accurate isotopic patterns underlying the isotope-pattern
    match scoring.
  repo: https://github.com/pyQms/pyqms
- name: Python
  role: Runtime environment (Python 3.7+) in which smiter.lib.csv_to_peak_properties,
    the noise injector, fragmentor, and smiter.synthetic_mzml.write_mzml are orchestrated.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:c433ab33ab115e582ab9f4e2cf20f0b645f3f91ff31018d51a179bd245387aea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  - pyQms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# isotope-pattern-match-scoring

## Summary

Score simulated LC-MS/MS runs (e.g. SMITER-generated mzML files) by matching observed isotope patterns against theoretical patterns to quantify how measurement error — such as a ppm mass offset — degrades quantification quality, as in Figure 5A/B of the SMITER paper.

## When to use

Use when comparing synthetic or real LC-MS/MS runs of known molecules (e.g. nucleosides Cm, m3C, m5C) to assess how mass-accuracy errors (ppm offsets) affect isotope-pattern match scores and quantification quality.

## When NOT to use

- The input run was not produced by a formula-based simulator like SMITER and there is no known ground truth (chemical formulas) to compute theoretical isotope patterns against.
- The goal is fragment-ion (MS2) identity rather than precursor isotope-pattern matching — SMITER's peptide/nucleoside fragmentation models address that separately.
- Only a single run with no error-free (0 ppm) counterpart exists, so the effect of the ppm offset on quantification quality cannot be assessed comparatively.

## Inputs

- Simulated (SMITER) or real LC-MS/MS run in mzML format
- Peak properties dict derived from the example_data CSV via smiter.lib.csv_to_peak_properties for the nucleosides Cm, m3C, m5C
- Instrument mass-accuracy offset setting (MACHINE_OFFSET_IN_PPM, e.g. 0 ppm and 4 ppm)

## Outputs

- Isotope-pattern match scores per spectrum for each simulated run
- Comparative quantification-quality pattern showing degradation at the 4 ppm offset relative to the 0 ppm offset (Figure 5A/B analogue)

## How to apply

First generate or obtain the mzML runs to be compared: for the SMITER sensitivity study, load the example_data CSV with peak properties of the modified nucleosides (Cm, m3C, m5C) via smiter.lib.csv_to_peak_properties, choose a nucleoside fragmentor and smiter.noise_functions.UniformNoiseInjector, set general params such as gradient_length, and run smiter.synthetic_mzml.write_mzml twice — once with MACHINE_OFFSET_IN_PPM = 0 and once with MACHINE_OFFSET_IN_PPM = 4. Then score isotope-pattern matches on both runs: isotope patterns are computed with highly-accurate isotopic patterns enabled by pyQms, and each spectrum receives an isotope-pattern match score. The expected outcome is that quantifications are optimal at 0 ppm, while a non-zero ppm offset produces a high number of spectra with low isotope-pattern match scores, reproducing the degradation pattern of Figure 5A/B.

## Related tools

- **SMITER** (Generates the synthetic mzML LC-MS/MS runs (0 ppm and 4 ppm MACHINE_OFFSET_IN_PPM) whose isotope-pattern matches are scored; isotope patterns are computed with pyQms.) — https://github.com/LeidelLab/SMITER
- **pyQms** (Provides the highly-accurate isotopic patterns underlying the isotope-pattern match scoring.) — https://github.com/pyQms/pyqms
- **Python** (Runtime environment (Python 3.7+) in which smiter.lib.csv_to_peak_properties, the noise injector, fragmentor, and smiter.synthetic_mzml.write_mzml are orchestrated.)

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('example_data.csv'); mzml_file = write_mzml(peak_properties, fragmentor, UniformNoiseInjector(), gradient_length=..., MACHINE_OFFSET_IN_PPM=4)
```

## Evaluation signals

- Both generated mzML files (0 ppm and 4 ppm runs) must be loadable.
- At 0 ppm offset, quantifications are optimal — isotope-pattern match scores should be uniformly high.
- At the 4 ppm offset, a high number of spectra should show low isotope-pattern match scores.
- The score distribution across the two runs must visibly reproduce the degradation pattern reported in Figure 5A/B.
- Identical simulation parameters (fragmentor, UniformNoiseInjector, gradient_length, peak properties) must be used for both runs so only the ppm offset differs.

## Limitations

- SMITER simulations depend on modular noise and fragmentation models; the default UniformNoiseInjector and nucleoside fragmentor may not reflect all instrument behaviours, so scored degradation is model-specific.
- The sensitivity result is demonstrated for the isomeric nucleosides Cm, m3C, m5C over a 0–4 ppm offset range; extrapolation to other offsets or analyte classes is not established in the paper.
- The README notes m/z- and intensity noise injection options (uniform noise or combined general + intensity-specific noise), so choice of noise model can affect match scores.

## Evidence

- [results] Increasing measurement imprecision degrades quantification quality: without ppm errors quantifications are optimal, while a non-zero ppm offset produces a high number of spectra with low isotope-pattern match scores.: [evidence span withheld — anchor status: not_found]
- [readme] usage of highly-accurate isotopic patterns enabled by `pyQms`_: [evidence span withheld — anchor status: external_doc]
- [methods] Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml`: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [readme] m/z-and intensity noise injection ( uniform noise or a noise model that combines general noise with intensity-specific noise): [evidence span withheld — anchor status: external_doc]
