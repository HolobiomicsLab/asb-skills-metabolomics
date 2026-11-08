---
name: simulation-runtime-benchmarking
description: Use when you must quantify how fast SMITER can synthesize a proteome-scale
  synthetic mzML (tens of thousands of peptides, thousands of MS2 scans) or verify
  that a simulated run meets a runtime and scan-count budget.
when_to_use_negative:
- The goal is to analyze or re-process an already-acquired real mzML file — SMITER
  only writes synthetic mzML from chemical formulas, it does not parse experimental
  runs.
- The target molecules require a fragmentation model not shipped with SMITER (it defaults
  to several peptide methods, two nucleoside models, and one lipid model) and there
  is no time to implement a new modular fragmentation model.
- The task is evaluating experimental MS data quality rather than pre-experiment simulation
  of co-elution and co-fragmentation hurdles.
tools:
- name: SMITER
  role: Python library whose csv_to_peak_properties, PeptideFragmentor, UniformNoiseInjector,
    and synthetic_mzml.write_mzml functions are executed and timed for the benchmark
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Runtime environment (README requires Python 3.7+ for LeidelLab/SMITER, 3.6+
    for MKoesters/SMITER) in which the benchmark script is run and wall-clock time
    is recorded
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:c20c12af4510ed1f258ff0c434d1d1e8d263acb4904c7bd95f0542b58a2fef32
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
yes_rate: 1.0
n_claims: 8
---

# simulation-runtime-benchmarking

## Summary

Benchmark SMITER's wall-clock runtime and output scale for a proteome-scale synthetic LC-MS/MS run: a 140-min yeast proteome gradient (36,854 peptides, top10 method, 11,640 MS2 scans) simulated in under three minutes on a standard laptop.

## When to use

Use when you must quantify how fast SMITER can synthesize a proteome-scale synthetic mzML (tens of thousands of peptides, thousands of MS2 scans) or verify that a simulated run meets a runtime and scan-count budget.

## When NOT to use

- The goal is to analyze or re-process an already-acquired real mzML file — SMITER only writes synthetic mzML from chemical formulas, it does not parse experimental runs.
- The target molecules require a fragmentation model not shipped with SMITER (it defaults to several peptide methods, two nucleoside models, and one lipid model) and there is no time to implement a new modular fragmentation model.
- The task is evaluating experimental MS data quality rather than pre-experiment simulation of co-elution and co-fragmentation hurdles.

## Inputs

- Peptide-list CSV (from example_data) containing chemical formulas and peak properties for 36,854 yeast proteome peptides
- Fragmentor configuration: fragmentation_functions.PeptideFragmentor with top10 fragmentation method
- Noise injector configuration: smiter.noise_functions.UniformNoiseInjector
- General simulation parameters dict with gradient_length = 140 (minutes)

## Outputs

- Synthetic mzML file containing 11,640 MS2 scans covering 36,854 peptides over a 140-min simulated gradient
- Wall-clock runtime measurement for the full simulation

## How to apply

Build the peak properties dict from a peptide-list CSV via smiter.lib.csv_to_peak_properties, using the yeast proteome list of 36,854 peptides from example_data as the benchmark input. Configure the fragmentor as fragmentation_functions.PeptideFragmentor with the top10 fragmentation method and smiter.noise_functions.UniformNoiseInjector as the noise generator. Set the general simulation parameter gradient_length to 140 minutes, then start a wall-clock timer and execute smiter.synthetic_mzml.write_mzml with the peak properties, fragmentor, noise injector, and general params. Validate by inspecting the output mzML for exactly 11,640 MS2 scans, coverage of all 36,854 peptides, and a total runtime under three minutes on a standard laptop.

## Related tools

- **SMITER** (Python library whose csv_to_peak_properties, PeptideFragmentor, UniformNoiseInjector, and synthetic_mzml.write_mzml functions are executed and timed for the benchmark) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime environment (README requires Python 3.7+ for LeidelLab/SMITER, 3.6+ for MKoesters/SMITER) in which the benchmark script is run and wall-clock time is recorded)

## Examples

```
peak_props = smiter.lib.csv_to_peak_properties('yeast_peptides.csv'); smiter.synthetic_mzml.write_mzml(peak_props, fragmentor=fragmentor, noise_injector=noise, params={'gradient_length': 140})
```

## Evaluation signals

- Output mzML contains exactly 11,640 MS2 scans
- The simulated run covers all 36,854 yeast peptides from the input CSV
- Total wall-clock runtime stays under three minutes on a standard laptop
- Gradient length in the written mzML corresponds to the 140-min parameter in general params
- Peak properties dict was successfully built from the CSV via csv_to_peak_properties (all peptides carry parseable chemical formulas)

## Limitations

- The <3 min threshold is anchored to 'a standard laptop'; runtime on other hardware will vary and the article provides no formal hardware spec beyond that phrase.
- Benchmark covers the top10 peptide fragmentation method with UniformNoiseInjector; other fragmentors or noise models (e.g., the combined general + intensity-specific noise model) were not timed in this benchmark and may scale differently.
- Discussion-section limitations were not extractable (no prose/changelog available), so scalability ceilings for larger proteomes or gradients are undocumented.

## Evidence

- [methods] csv_to_peak_properties builds peak properties: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
- [methods] write_mzml is the execution entry point: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [abstract] SMITER purpose: "SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs."
- [readme] chemical-formula abstraction enables any biomolecule: "Python library to create synthetic mzMLs file based on chemical formulas. All molecules can be simulated du to abstraction to chemical formulas."
