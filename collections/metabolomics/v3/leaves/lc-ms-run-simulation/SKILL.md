---
name: lc-ms-run-simulation
description: Use when you need gold-standard synthetic mzML datasets with known ground
  truth — e.g. to benchmark quantification algorithms, test mass-accuracy (ppm) sensitivity,
  or predict co-elution/co-fragmentation before real MS experiments.
when_to_use_negative:
- The input is already a real experimental mzML measurement rather than a set of analyte
  formulas to simulate
- You need validated quantification of real biological samples — SMITER creates gold-standard
  synthetic data, not measurement analysis
- The analyte class lacks a suitable default fragmentation model and you do not intend
  to implement a custom one
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMITER
  role: 'Core simulation library: csv_to_peak_properties, fragmentors, noise injectors,
    and smiter.synthetic_mzml.write_mzml'
  repo: https://github.com/LeidelLab/SMITER
- name: pyQms
  role: Provides highly-accurate isotopic pattern calculation for simulated molecules
  repo: https://github.com/pyQms/pyqms
- name: Python
  role: Runtime environment (Python 3.7+ required); simulation driven via Python API
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:4d40d556434d8b2fe93d4430457d0226c726e3f1bb2cf52603e78dcaafdf87a6
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# lc-ms-run-simulation

## Summary

Simulate complete synthetic LC-MS/MS runs of biomolecules (peptides, modified nucleosides, lipids) with SMITER by converting chemical formulas into peak properties, applying a fragmentation model and noise injector, and writing a synthetic mzML file.

## When to use

Use when you need gold-standard synthetic mzML datasets with known ground truth — e.g. to benchmark quantification algorithms, test mass-accuracy (ppm) sensitivity, or predict co-elution/co-fragmentation before real MS experiments.

## When NOT to use

- The input is already a real experimental mzML measurement rather than a set of analyte formulas to simulate
- You need validated quantification of real biological samples — SMITER creates gold-standard synthetic data, not measurement analysis
- The analyte class lacks a suitable default fragmentation model and you do not intend to implement a custom one

## Inputs

- CSV file of analyte peak properties (chemical formulas, retention times) from example_data
- Fragmentor choice (e.g. nucleoside fragmentation model, PeptideFragmentor)
- Noise injector configuration (e.g. UniformNoiseInjector with 1000 counts white noise)
- General simulation parameters: gradient_length, scan time difference, m, y, MACHINE_OFFSET_IN_PPM

## Outputs

- Synthetic mzML file of the full simulated LC-MS/MS run
- TIC extractable from the simulated mzML for comparison against real measurements

## How to apply

Build the peak properties dict from a CSV of analyte formulas and retention times via smiter.lib.csv_to_peak_properties (using e.g. Modomics reference retention times for nucleosides). Choose a fragmentor appropriate to the analyte class — e.g. a nucleoside fragmentation model (optionally with 10% fragment omission) or fragmentation_functions.PeptideFragmentor — and a noise generator such as smiter.noise_functions.UniformNoiseInjector (e.g. 1000 counts white noise). Set general parameters including gradient_length (e.g. 80 min), scan time difference (e.g. 60 ms), and the m/y peak-shape parameters (e.g. m=0.001701, y=0.2); optionally set MACHINE_OFFSET_IN_PPM (e.g. 0 vs 4 ppm) to inject mass-accuracy error. Run the simulation with smiter.synthetic_mzml.write_mzml, passing the peak properties, fragmentor, and noise injector, to produce the synthetic mzML. Isotopic patterns are computed with pyQms, and peak shapes are scaled with gauss, gamma, or exponentially-modified gaussian distributions.

## Related tools

- **SMITER** (Core simulation library: csv_to_peak_properties, fragmentors, noise injectors, and smiter.synthetic_mzml.write_mzml) — https://github.com/LeidelLab/SMITER
- **pyQms** (Provides highly-accurate isotopic pattern calculation for simulated molecules) — https://github.com/pyQms/pyqms
- **Python** (Runtime environment (Python 3.7+ required); simulation driven via Python API)

## Examples

```
from smiter.lib import csv_to_peak_properties; from smiter.synthetic_mzml import write_mzml; peak_properties = csv_to_peak_properties('nucleoside_mix.csv'); write_mzml(peak_properties, fragmentor, noise_injector, gradient_length=80)
```

## Evaluation signals

- The written mzML loads and its TIC shows peaks for all injected analytes at the specified retention times
- Simulated TIC contains the injected noise level (e.g. basic white noise at 1000 counts) — note simulated runs typically exhibit LOWER noise than comparable real measurements
- With MACHINE_OFFSET_IN_PPM=0, isotope-pattern match scores are optimal; at 4 ppm a high number of spectra with low match scores appears (Figure 5A/B behavior)
- Known absence of unmodeled features — e.g. the simulated 24-nucleoside run lacks the real measurement's unidentified ~15 min contaminant peak
- Package installation verified with tox

## Limitations

- Simulated runs exhibit lower noise than real measurements and do not reproduce unmodeled contaminants (e.g. the unidentified ~15 min peak in the real nucleoside standard measurement)
- Default fragmentation models cover peptides, two nucleoside models, and one lipid model; other analyte classes require custom implementation, though the modular design allows this
- Discussion-section limitations are unavailable from the extracted text (no changelog/prose found)

## Evidence

- [abstract] SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate LC-MS/MS runs.: "SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate LC-MS/MS runs."
- [methods] 5. Run the simulation and write the resulting mzML using smiter.synthetic_mzml.write_mzml: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [readme] usage of highly-accurate isotopic patterns enabled by pyQms: [evidence span withheld — anchor status: external_doc]
- [readme] SMITER requires Python 3.7 or higher.: [evidence span withheld — anchor status: external_doc]
