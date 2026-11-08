---
name: peptide-fragmentation-simulation
description: Use when you need a gold-standard synthetic proteomics mzML — e.g. to
  benchmark search/scoring algorithms, predict co-elution/co-fragmentation, or compare
  simulated vs. real TIC behavior for a peptide list on a defined gradient length.
when_to_use_negative:
- The input is already real acquired LC-MS/MS data to be identified or quantified
  — SMITER generates synthetic mzML, it does not analyze measurements.
- The target molecules are not representable by a chemical formula, since SMITER's
  abstraction requires formulas for all calculations.
- Only a retention-time prediction for a peptide list is needed without a full LC-MS/MS
  simulation — RT prediction is a pluggable module rather than this skill's core workflow.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SMITER
  role: 'Core library: csv_to_peak_properties, PeptideFragmentor, UniformNoiseInjector,
    and write_mzml together generate the synthetic peptide mzML'
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: 'Runtime environment for the simulation script (reference implementation:
    complex_mix_simulation.py); requires Python 3.6/3.7+'
  repo: https://pypi.python.org/pypi/smiter
- name: pyQms
  role: Provides highly-accurate isotopic pattern calculations underlying the simulated
    peptide peaks
  repo: https://github.com/pyQms/pyqms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:95170e49003f1ba825538b1f38f14445fda53e8d636e064b428157d53163b0e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  - pyQms
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# peptide-fragmentation-simulation

## Summary

Simulate a full peptide LC-MS/MS run with SMITER by converting an identified peptide list into a peak-properties dict, fragmenting MS2 spectra with PeptideFragmentor, injecting noise, and writing a synthetic mzML whose TIC can be compared against a real acquisition (e.g. the 140-min Bruderer et al. proteomics gradient).

## When to use

Use when you need a gold-standard synthetic proteomics mzML — e.g. to benchmark search/scoring algorithms, predict co-elution/co-fragmentation, or compare simulated vs. real TIC behavior for a peptide list on a defined gradient length.

## When NOT to use

- The input is already real acquired LC-MS/MS data to be identified or quantified — SMITER generates synthetic mzML, it does not analyze measurements.
- The target molecules are not representable by a chemical formula, since SMITER's abstraction requires formulas for all calculations.
- Only a retention-time prediction for a peptide list is needed without a full LC-MS/MS simulation — RT prediction is a pluggable module rather than this skill's core workflow.

## Inputs

- CSV file of identified peptides with chemical formulas (e.g. peptide list from the Bruderer et al. dataset)
- peak properties dict produced by smiter.lib.csv_to_peak_properties
- general parameters: gradient_length (e.g. 140 min), scan interval (e.g. 60 ms)

## Outputs

- Synthetic mzML file (e.g. complex_mix_simulation.mzml) containing simulated MS1/MS2 peptide scans with injected noise
- Total ion chromatogram (TIC) for comparison against a real LC-MS/MS measurement

## How to apply

Build the peak properties dict from a peptide CSV via smiter.lib.csv_to_peak_properties, including each peptide's chemical formula (SMITER computes everything from formulas, with isotopic patterns from pyQms). Choose fragmentation_functions.PeptideFragmentor for MS2 fragmentation and smiter.noise_functions.UniformNoiseInjector as the noise generator. Define general parameters such as gradient_length (e.g. 140 min for the Bruderer-style run) and scan interval (e.g. 60 ms). Call smiter.synthetic_mzml.write_mzml passing peak_properties, the noise injector, and the fragmentor to produce e.g. complex_mix_simulation.mzml. Judge correctness by comparing the simulated total ion chromatogram to the real measurement: the simulation should show more distinct features and a constantly increasing TIC over the long RT window without major intensity drops, whereas the real run fluctuates — this divergence is the expected, reported outcome, not a failure.

## Related tools

- **SMITER** (Core library: csv_to_peak_properties, PeptideFragmentor, UniformNoiseInjector, and write_mzml together generate the synthetic peptide mzML) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime environment for the simulation script (reference implementation: complex_mix_simulation.py); requires Python 3.6/3.7+) — https://pypi.python.org/pypi/smiter
- **pyQms** (Provides highly-accurate isotopic pattern calculations underlying the simulated peptide peaks) — https://github.com/pyQms/pyqms

## Examples

```
peak_properties = smiter.lib.csv_to_peak_properties('peptides.csv'); smiter.synthetic_mzml.write_mzml(peak_properties, fragmentor=smiter.fragmentation_functions.PeptideFragmentor(), noise_injector=smiter.noise_functions.UniformNoiseInjector(), gradient_length=140)
```

## Evaluation signals

- The written mzML was produced by smiter.synthetic_mzml.write_mzml with the peptide peak_properties, fragmentor, and noise injector supplied as parameters.
- The simulated TIC rises constantly over a long RT window with no major intensity drops, matching the reported Figure 4 behavior.
- The simulated run shows more distinct features than the real Bruderer measurement, per the reported comparison.
- Gradient and scan parameters are respected: gradient_length = 140 min and scan time difference of 60 ms in the generated scans.
- MS2 spectra are produced from the peptides under the PeptideFragmentor, not generic fragments, and uniform noise is present in the baseline.

## Limitations

- The simulated TIC does not reproduce real-measurement behavior — the real Bruderer run fluctuates and does not steadily increase, while the simulation steadily rises, so the model omits major intensity drops.
- No changelog found, and the discussion section content is absent, so limitations and edge cases are not formally documented in the source article.
- Only a uniform noise injector is referenced in the peptide workflow; the README lists richer noise options (general + intensity-specific noise) but the proteomics example uses UniformNoiseInjector.

## Evidence

- [methods] write_mzml call and parameters: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [readme] formula-based simulation scope: "It enables the simulation of any biomolecule since all calculations are based on the chemical formulas."
- [readme] isotopic pattern source: [evidence span withheld — anchor status: external_doc]
- [results] expected simulated-vs-real TIC divergence: [evidence span withheld — anchor status: not_found]
- [abstract] gold-standard rationale: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
