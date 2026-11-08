---
name: nucleoside-fragmentation-simulation
description: Use when you need gold-standard synthetic mzML data for modified nucleosides
  whose isomeric forms are hard to distinguish, or to test how instrument mass-accuracy
  error (ppm offsets) degrades isotope-pattern-based quantification before running
  real MS experiments.
when_to_use_negative:
- Your molecules are unmodified peptides for which SMITER's default PeptideFragmentor
  is the intended fragmentation model rather than a nucleoside fragmentor
- You need to process real, measured mzML/LC-MS data rather than generate synthetic
  gold-standard runs (SMITER writes synthetic mzML, it does not analyze acquired data)
- You lack chemical formulas for the target biomolecules, since SMITER's simulation
  is entirely formula-based
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMITER
  role: Python library that simulates the LC-MS/MS runs of the isomeric nucleosides
    and writes the synthetic mzML files via smiter.synthetic_mzml.write_mzml
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Runtime language for the simulation scripts, including csv_to_peak_properties
    conversion and setting MACHINE_OFFSET_IN_PPM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:aef8f11498f915ad6b71a305a58812b684bb0e521d0eaa30fa611bd90968091a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nucleoside-fragmentation-simulation

## Summary

Simulates LC-MS/MS runs of modified nucleosides (e.g. the isomeric tri-methylated cytidines Cm, m3C, m5C) with SMITER by converting peak properties from chemical formulas, applying a nucleoside fragmentor and noise injector, and writing synthetic mzML files at controlled mass-accuracy offsets (e.g. 0 vs 4 ppm) to benchmark quantification quality.

## When to use

Use when you need gold-standard synthetic mzML data for modified nucleosides whose isomeric forms are hard to distinguish, or to test how instrument mass-accuracy error (ppm offsets) degrades isotope-pattern-based quantification before running real MS experiments.

## When NOT to use

- Your molecules are unmodified peptides for which SMITER's default PeptideFragmentor is the intended fragmentation model rather than a nucleoside fragmentor
- You need to process real, measured mzML/LC-MS data rather than generate synthetic gold-standard runs (SMITER writes synthetic mzML, it does not analyze acquired data)
- You lack chemical formulas for the target biomolecules, since SMITER's simulation is entirely formula-based

## Inputs

- CSV file of peak properties for modified nucleosides (example_data, containing Cm, m3C, m5C entries)
- Chemical formulas of the nucleosides (SMITER abstracts all molecules to chemical formulas)
- Simulation configuration: fragmentor choice, noise injector (e.g. UniformNoiseInjector), general params such as gradient_length
- Instrument mass-accuracy offset setting MACHINE_OFFSET_IN_PPM (e.g. 0 ppm and 4 ppm)

## Outputs

- Two synthetic mzML files: one 0-ppm simulated run and one 4-ppm simulated run
- Isotope-pattern match scores / quantification-quality comparison showing degradation at the non-zero ppm offset

## How to apply

Load the example_data CSV with peak properties of the modified nucleosides and convert it to a peak properties dict using smiter.lib.csv_to_peak_properties. Configure a nucleoside fragmentor, a noise generator such as smiter.noise_functions.UniformNoiseInjector, and general simulation parameters like gradient_length. Run the simulation twice: once with the instrument mass offset MACHINE_OFFSET_IN_PPM set to 0 ppm and once at 4 ppm, each time writing the result with smiter.synthetic_mzml.write_mzml. The rationale is that all calculations are based on chemical formulas, so isomeric nucleosides differing only in modification position are simulated with identical formulas but distinguishable via isotope-pattern match scores; the 0 ppm run serves as the optimal-quantification baseline against which degradation at 4 ppm is judged. Validate by confirming both mzML files load and that isotope-pattern scoring reproduces the reported pattern of many low-match-score spectra at the 4 ppm offset versus optimal quantification at 0 ppm.

## Related tools

- **SMITER** (Python library that simulates the LC-MS/MS runs of the isomeric nucleosides and writes the synthetic mzML files via smiter.synthetic_mzml.write_mzml) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime language for the simulation scripts, including csv_to_peak_properties conversion and setting MACHINE_OFFSET_IN_PPM)

## Examples

```
from smiter.lib import csv_to_peak_properties; from smiter.synthetic_mzml import write_mzml; peak_properties = csv_to_peak_properties('example_data.csv'); write_mzml(peak_properties, fragmentor, noise_injector, MACHINE_OFFSET_IN_PPM=4)
```

## Evaluation signals

- Both generated mzML files must be loadable by standard mzML parsers
- The 0-ppm run should yield optimal quantification (high isotope-pattern match scores), matching the reported baseline
- The 4-ppm run should show a high number of spectra with low isotope-pattern match scores, reproducing the Figure 5A/B degradation pattern
- The peak properties dict must contain the expected formula-based entries for Cm, m3C and m5C after csv_to_peak_properties conversion
- Only the MACHINE_OFFSET_IN_PPM setting should differ between the two simulated runs; all other parameters (fragmentor, noise injector, gradient_length) must be identical

## Limitations

- Non-zero ppm offsets produce a high number of spectra with low isotope-pattern match scores, so quantification quality is strongly dependent on mass accuracy
- SMITER's default noise and fragmentation models are fixed choices (e.g. UniformNoiseInjector); other models require implementing them into its modular design
- The article's discussion section provides no changelog or limitation prose, so additional caveats could not be extracted

## Evidence

- [readme] All molecules can be simulated du to abstraction to chemical formulas.: "Python library to create synthetic mzMLs file based on chemical formulas. All molecules can be simulated du to abstraction to chemical formulas."
- [methods] workflow step: csv to peak properties dict: "1. create the peak properties dict (You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`)"
- [methods] workflow step: write mzML: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [abstract] gold-standard datasets motivation: "This allows for the facile creation of defined gold-standard-LC-MS/MS datasets for any type of experiment."
- [abstract] modular noise/fragmentation design: "As SMITER features a modular design, noise and fragmentation models can easily be implemented or adapted."
