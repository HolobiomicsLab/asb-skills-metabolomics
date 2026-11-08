---
name: lc-msms-dda-run-simulation
description: Use when a gold-standard LC-MS/MS dataset with known ground truth is
  needed to benchmark peptide identification pipelines, test search/rescoring algorithms,
  or predict co-elution and co-fragmentation before running real MS experiments.
when_to_use_negative:
- The input is already an experimental LC-MS/MS dataset — SMITER simulates runs from
  chemical formulas and is for generating gold standards, not re-analyzing measured
  data.
- You need simulation of acquisition modes or fragmentation chemistries outside SMITER's
  default models (several peptide fragmentation methods, two nucleoside models, one
  lipid model) without implementing a custom module.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMITER
  role: Core Python library that writes the synthetic mzML from peak properties, fragmentor,
    noise injector, and general params
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Runtime environment and scripting language for driving the SMITER API (requires
    Python 3.7+ per README)
  repo: https://pypi.python.org/pypi/smiter
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:b40840ae83c8552f4e5af3e2c4e03522e05cc03b5b7c17223efa9f3996bed3f4
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-msms-dda-run-simulation

## Summary

Simulate synthetic LC-MS/MS (DDA) runs with SMITER, a Python library that builds mzML files from chemical formulas for arbitrary biomolecules, enabling gold-standard datasets with known ground truth for benchmarking search pipelines.

## When to use

Use when a gold-standard LC-MS/MS dataset with known ground truth is needed to benchmark peptide identification pipelines, test search/rescoring algorithms, or predict co-elution and co-fragmentation before running real MS experiments.

## When NOT to use

- The input is already an experimental LC-MS/MS dataset — SMITER simulates runs from chemical formulas and is for generating gold standards, not re-analyzing measured data.
- You need simulation of acquisition modes or fragmentation chemistries outside SMITER's default models (several peptide fragmentation methods, two nucleoside models, one lipid model) without implementing a custom module.

## Inputs

- CSV of peptides (with chemical formulas) as in the SMITER example_data
- In silico trypsin-digested proteome (e.g. S. cerevisiae) for defining simulated peptide sets
- DeepLC retention time predictions for the peptides
- Simulation parameters (gradient_length, peak width, dynamic exclusion, MS2 scans per cycle, DDA topN method)

## Outputs

- Synthetic mzML file(s), one per simulated peptide set
- Identification rate curve (e.g. identification_rate_curve.png) plotting PSMs at 1% q-value versus peptide set size
- Wall-clock runtime and MS2 scan counts for benchmarking

## How to apply

Build the peak properties dict from a peptide list CSV via smiter.lib.csv_to_peak_properties, predicting retention times with DeepLC. Choose a fragmentor (e.g. fragmentation_functions.PeptideFragmentor with a topN method such as top10/top20 DDA), a noise generator (smiter.noise_functions.UniformNoiseInjector), and general params such as gradient_length (e.g. 140 min), peak width 30–120 s, dynamic exclusion 30 s, minimum intensity 500, and at most 20 MS2 scans per MS1 cycle. Run smiter.synthetic_mzml.write_mzml with these components to produce one mzML per simulated peptide set. Search the synthetic mzML (e.g. with X!Tandem against the yeast proteome), rescore with Percolator, filter PSMs at 1% q-value, and compare identification rate to the known simulated peptides to validate the simulation. Benchmark checks: a 5,000-peptide set should recover ~90% of peptides, and a proteome-scale run (36,854 peptides, 140-min gradient, top10 method) should yield ~11,640 MS2 scans in under three minutes on a standard laptop.

## Related tools

- **SMITER** (Core Python library that writes the synthetic mzML from peak properties, fragmentor, noise injector, and general params) — https://github.com/LeidelLab/SMITER
- **Python** (Runtime environment and scripting language for driving the SMITER API (requires Python 3.7+ per README)) — https://pypi.python.org/pypi/smiter

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('example_data.csv'); smiter.synthetic_mzml.write_mzml(peak_properties, fragmentor=PeptideFragmentor(), noise_injector=UniformNoiseInjector(), params={'gradient_length': 140})
```

## Evaluation signals

- Output mzML is well-formed and contains the expected MS2 scan count (e.g. 11,640 MS2 scans for a 36,854-peptide top10 run with 140-min gradient)
- For a small simulated set (~5,000 peptides), X!Tandem + Percolator searching at 1% q-value recovers ~90% of simulated peptides, most via multiple PSMs
- Proteome-scale simulation wall-clock runtime stays under three minutes on a standard laptop
- Each mzML corresponds to exactly one peptide set size, enabling a monotone identification-rate-versus-set-size curve comparable to Figure 6

## Limitations

- Identification rate degrades as peptide set size grows — small sets (~5,000 peptides) recover ~90%, whereas sets over 130,000 peptides drop below 50% — so simulations must be matched to the intended question.
- Default fragmentation coverage is limited to several peptide methods, two nucleoside models, and one lipid model; other molecule classes require implementing custom noise/fragmentation modules.
- No changelog was available in the source context, so version-specific behavior changes cannot be traced from the article alone.

## Evidence

- [abstract] SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs.: "SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs."
- [methods] Run the simulation and write the resulting mzML using smiter.synthetic_mzml.write_mzml: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [methods] csv_to_peak_properties converts example_data CSV into the peak properties dict: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
- [readme] SMITER features include isotopic patterns via pyQms, feature scaling distributions, and noise injection: [evidence span withheld — anchor status: external_doc]
