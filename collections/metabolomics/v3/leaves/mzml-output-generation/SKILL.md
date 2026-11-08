---
name: mzml-output-generation
description: Use when you need a synthetic mzML gold-standard for benchmarking MS
  algorithms, predicting co-elution/co-fragmentation, or simulating runs for arbitrary
  biomolecules given only chemical formulas.
when_to_use_negative:
- Input analytes are not representable as chemical formulas, so isotope-pattern-based
  simulation is impossible
- The goal is to process or search real experimental spectra rather than create ground-truth
  simulated data
- A fragmentation or noise behavior is required that is not implemented and cannot
  be adapted within SMITER's modular noise/fragmentation interfaces
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMITER
  role: Python library whose write_mzml routine generates the synthetic mzML from
    peak properties, fragmentor, and noise injector
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Runtime environment (3.7+ per LeidelLab README); install via pip install smiter
    or from source with python setup.py install
- name: pyQms
  role: Provides highly-accurate isotopic pattern calculations underlying the simulated
    molecule signals
  repo: https://github.com/pyQms/pyqms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:e083c72509068bd87c5770a9f6bff9fccb5f4353002aff666326b805c9bb4ace
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mzml-output-generation

## Summary

Generate synthetic LC-MS/MS mzML files with SMITER by building a peak-properties dict from a peptide CSV, choosing a fragmentor and noise injector, and writing the simulated run via smiter.synthetic_mzml.write_mzml. This produces gold-standard datasets where ground truth is known for benchmarking computational MS algorithms.

## When to use

Use when you need a synthetic mzML gold-standard for benchmarking MS algorithms, predicting co-elution/co-fragmentation, or simulating runs for arbitrary biomolecules given only chemical formulas.

## When NOT to use

- Input analytes are not representable as chemical formulas, so isotope-pattern-based simulation is impossible
- The goal is to process or search real experimental spectra rather than create ground-truth simulated data
- A fragmentation or noise behavior is required that is not implemented and cannot be adapted within SMITER's modular noise/fragmentation interfaces

## Inputs

- CSV file of molecules/peptides (e.g. from example_data) with chemical formulas
- Fragmentation method choice (e.g. top10 for peptides)
- Noise model selection (e.g. UniformNoiseInjector)
- General simulation parameters dict (e.g. gradient_length = 140 min)

## Outputs

- Synthetic mzML file containing simulated MS2 scans (e.g. 11,640 MS2 scans for a 36,854-peptide yeast proteome run)

## How to apply

Build the peak properties dict from a peptide list CSV using smiter.lib.csv_to_peak_properties (as in example_data). Choose a fragmentor, e.g. fragmentation_functions.PeptideFragmentor with a fragmentation method such as top10 for peptides, and a noise generator such as smiter.noise_functions.UniformNoiseInjector. Define general params including gradient_length (e.g. 140 min for a proteome-scale yeast run). Execute the simulation with smiter.synthetic_mzml.write_mzml, passing peak_properties, fragmentor, noise injector, and general params. Isotopic patterns come from pyQms, and feature scaling uses gauss-, gamma-, or exponentially-modified gaussian distributions; the modular design means noise and fragmentation models can be swapped or extended.

## Related tools

- **SMITER** (Python library whose write_mzml routine generates the synthetic mzML from peak properties, fragmentor, and noise injector) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime environment (3.7+ per LeidelLab README); install via pip install smiter or from source with python setup.py install)
- **pyQms** (Provides highly-accurate isotopic pattern calculations underlying the simulated molecule signals) — https://github.com/pyQms/pyqms

## Examples

```
from smiter.lib import csv_to_peak_properties; from smiter.synthetic_mzml import write_mzml; peak_props = csv_to_peak_properties('example_data.csv'); write_mzml(peak_props, fragmentor, noise_injector, {'gradient_length': 140})
```

## Evaluation signals

- Output mzML contains the expected number of MS2 scans (e.g. 11,640 scans for the top10 yeast run)
- The simulated run covers all input molecules (e.g. 36,854 yeast peptides)
- Wall-clock runtime is plausible for the scale (proteome-scale top10 run completes in <3 min on a standard laptop)
- Resulting mzML parses as a valid file readable by standard mzML tooling and reflects the configured gradient_length
- Installation verified by running tox, which tests the package and correct installation

## Limitations

- Runtime scales with dataset size; proteome-scale runs (36,854 peptides, 140-min gradient) approach minutes of wall time on a standard laptop
- Only fragmentation models shipped by default are available out of the box (several peptide methods, two nucleoside models, one lipid model); others require custom implementation
- Two README variants differ in stated Python requirements (3.6+ in MKoesters fork vs 3.7+ in LeidelLab)

## Evidence

- [methods] Workflow: write mzML with fragmentor and noise injector: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [methods] csv_to_peak_properties conversion: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
- [abstract] Formula-based simulation scope: "It enables the simulation of any biomolecule amenable to mass spectrometry (MS) since all calculations are based on chemical formulas."
- [readme] pyQms isotopic patterns: [evidence span withheld — anchor status: external_doc]
- [readme] Gold-standard rationale: [evidence span withheld — anchor status: external_doc]
- [results] Proteome-scale runtime and scale finding: [evidence span withheld — anchor status: not_found]
