---
name: nucleoside-fragmentation-modeling
description: Use when you need a synthetic mzML ground-truth LC-MS/MS run of modified
  nucleoside standards — e.g. to benchmark identification/quantification algorithms
  or compare simulated vs. real TIC and noise profiles.
when_to_use_negative:
- You need fragmentation modelling for molecule classes outside SMITER's default scope
  — while SMITER can in principle simulate any biomolecule via chemical formulas,
  default fragmentors cover peptides, modified nucleosides, and lipids only
- You need realistic noise levels matching a real instrument run — the simulated run
  exhibits lower noise than the corresponding real measurement and only contains the
  deliberately injected white noise
- Your input is raw vendor data (e.g. raw Thermo/Agilent files) rather than chemical
  formulas/retention-time annotations — SMITER builds everything from chemical formulas
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SMITER
  role: Python library providing csv_to_peak_properties, nucleoside fragmentation
    models, noise injectors, and smiter.synthetic_mzml.write_mzml to generate the
    synthetic mzML
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: runtime environment for scripting the simulation pipeline (requires Python
    3.7 or higher)
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:8752c5b80399bd0423fc8b73eaadee6fe8d6b61e3509fa877fee67406c5164ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nucleoside-fragmentation-modeling

## Summary

Simulates synthetic LC-MS/MS runs of (modified) nucleoside standards with SMITER by converting a standard-mix csv into a peak-properties dict, applying a nucleoside MS2 fragmentor and a noise injector (e.g. UniformNoiseInjector with 1000 counts of white noise), and writing the result as an mzML file via smiter.synthetic_mzml.write_mzml.

## When to use

Use when you need a synthetic mzML ground-truth LC-MS/MS run of modified nucleoside standards — e.g. to benchmark identification/quantification algorithms or compare simulated vs. real TIC and noise profiles.

## When NOT to use

- You need fragmentation modelling for molecule classes outside SMITER's default scope — while SMITER can in principle simulate any biomolecule via chemical formulas, default fragmentors cover peptides, modified nucleosides, and lipids only
- You need realistic noise levels matching a real instrument run — the simulated run exhibits lower noise than the corresponding real measurement and only contains the deliberately injected white noise
- Your input is raw vendor data (e.g. raw Thermo/Agilent files) rather than chemical formulas/retention-time annotations — SMITER builds everything from chemical formulas

## Inputs

- csv file of the nucleoside standard mix (e.g. from SMITER's example_data) with chemical formulas and Modomics reference retention times
- fragmentor selection: one of SMITER's two nucleoside fragmentation models
- noise injector instance, e.g. smiter.noise_functions.UniformNoiseInjector
- general parameters dict: gradient_length (e.g. 80 min = 4800 s), scan time difference (e.g. 60 ms), m (0.001701), y (0.2)

## Outputs

- synthetic LC-MS/MS run written as an mzML file containing MS1 isotopic patterns and MS2 nucleoside fragment spectra plus injected noise
- TIC extractable from the simulated mzML for comparison against real measurements

## How to apply

Load the nucleoside standard mix csv from SMITER's example_data and build the peak properties dict with smiter.lib.csv_to_peak_properties, using Modomics reference retention times. Select one of the two nucleoside fragmentation models as the fragmentor (with ~10% fragment omission as in the paper's complex-mix example), and choose a noise generator such as smiter.noise_functions.UniformNoiseInjector (e.g. 1000 counts of white noise). Set general parameters — gradient_length 80 min, scan time difference 60 ms, m=0.001701, y=0.2 — then run the simulation with smiter.synthetic_mzml.write_mzml, passing peak_properties, fragmentor, and noise injector. Extract the TIC from the resulting mzML to compare against a real measurement, expecting lower noise in the simulation, absence of unmodelled contaminant peaks (e.g. a ~15 min contaminant in the real data), and only the injected white noise.

## Related tools

- **SMITER** (Python library providing csv_to_peak_properties, nucleoside fragmentation models, noise injectors, and smiter.synthetic_mzml.write_mzml to generate the synthetic mzML) — https://github.com/LeidelLab/SMITER
- **Python** (runtime environment for scripting the simulation pipeline (requires Python 3.7 or higher)) — https://www.python.org/downloads/

## Examples

```
from smiter.lib import csv_to_peak_properties; from smiter.synthetic_mzml import write_mzml; from smiter.noise_functions import UniformNoiseInjector; peak_properties = csv_to_peak_properties('nucleoside_standard_mix.csv'); mzml = write_mzml('sim_run.mzML', peak_properties, fragmentor=fragmentor, noise_injector=UniformNoiseInjector(), params={'gradient_length': 4800, 'scan_time_dict': {'charge': {'default': 60000}}})
```

## Evaluation signals

- The write_mzml call completes and produces a valid mzML file whose gradient length matches the configured value (e.g. 80 min at 60 ms scan time difference).
- Extracted TIC from the simulated mzML contains the expected nucleoside peaks at their Modomics reference retention times and shows only the injected white noise (e.g. 1000 counts UniformNoiseInjector).
- Simulated-vs-real TIC comparison reproduces the paper's Figure 3 observations: lower noise in the simulation and absence of the unidentified ~15 min contaminant peak present in the real measurement.
- MS2 spectra reflect the chosen nucleoside fragmentor, including the specified fraction of omitted fragments (~10%).
- Installation sanity check passes via the package's own test suite (tox).

## Limitations

- The simulated run exhibits lower noise levels than the corresponding real measurement of the 24 nucleoside standards mix.
- Simulations lack real-world artefacts such as the unidentified contaminant peak around 15 min seen in the real data.
- Default nucleoside fragmentation support is limited to two models; noise modelling beyond the default (uniform or intensity-specific) requires implementing new modular noise generators.
- The README notes SMITER requires a virtual environment and Python 3.7+ (LeidelLab) / 3.6+ (MKoesters dev repo); no changelog is available.

## Evidence

- [abstract] SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models.: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [methods] choose fragmentor: "2. Choose a fragmentor (e.g. fragmentation_functions.PeptideFragmentor)"
- [methods] choose noise generator: "3. Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [methods] write mzml: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [methods] convert csv: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
- [readme] readme features: [evidence span withheld — anchor status: external_doc]
- [readme] readme noise injection: [evidence span withheld — anchor status: external_doc]
