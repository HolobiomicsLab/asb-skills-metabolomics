---
name: mzml-file-generation
description: Use when a ground-truth LC-MS/MS dataset with known molecule identities,
  abundances, and retention times is needed to benchmark quantification or identification
  algorithms before running real MS experiments.
when_to_use_negative:
- The input is already measured LC-MS/MS data and the goal is analysis or identification
  rather than creation of a synthetic ground-truth benchmark
- The molecules of interest cannot be represented as chemical formulas, so SMITER's
  formula-based isotope-pattern and fragmentation calculations do not apply
- A realistic noise/contaminant profile identical to a specific instrument is required
  — simulated runs exhibit lower noise than real measurements and lack unidentified
  contaminant peaks (e.g. the ~15 min peak in the real nucleoside measurement)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMITER
  role: Core library whose smiter.synthetic_mzml.write_mzml function runs the simulation
    and writes the synthetic mzML; fragmentors and noise injectors are supplied as
    modular components
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Language in which the simulation script (e.g. complex_mix_simulation.py) is
    written and executed; SMITER is installed via pip or setup.py
  repo: https://pypi.python.org/pypi/smiter
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:8cb78c0fc8a576aa75f27475cee433c583149982fd6e50fbdbf746c229a67879
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mzml-file-generation

## Summary

Generate synthetic LC-MS/MS mzML files with SMITER by simulating chromatographic peaks, isotope patterns, MS2 fragmentation, and noise from chemical formulas, producing gold-standard datasets with known ground truth.

## When to use

Use when a ground-truth LC-MS/MS dataset with known molecule identities, abundances, and retention times is needed to benchmark quantification or identification algorithms before running real MS experiments.

## When NOT to use

- The input is already measured LC-MS/MS data and the goal is analysis or identification rather than creation of a synthetic ground-truth benchmark
- The molecules of interest cannot be represented as chemical formulas, so SMITER's formula-based isotope-pattern and fragmentation calculations do not apply
- A realistic noise/contaminant profile identical to a specific instrument is required — simulated runs exhibit lower noise than real measurements and lack unidentified contaminant peaks (e.g. the ~15 min peak in the real nucleoside measurement)

## Inputs

- CSV file of peak properties (chemical formulas, retention times, intensities) e.g. from the SMITER example_data directory
- Fragmentor choice (e.g. PeptideFragmentor, nucleoside fragmentation model, lipid fragmentation model)
- Noise generator configuration (e.g. UniformNoiseInjector with 1000 counts white noise)
- General simulation parameters: gradient_length (min), scan time difference (e.g. 60 ms), MACHINE_OFFSET_IN_PPM

## Outputs

- Synthetic mzML file (e.g. complex_mix_simulation.mzml) containing the simulated LC-MS/MS run with MS1 and MS2 spectra
- Gold-standard dataset with known ground-truth molecule identities, abundances, and elution windows

## How to apply

Build the peak properties dict from a CSV of molecule formulas, retention times (e.g. Modomics reference RTs for nucleosides), and intensities using smiter.lib.csv_to_peak_properties. Choose a fragmentor matching the molecule class (e.g. fragmentation_functions.PeptideFragmentor for peptides, a nucleoside fragmentation model with a chosen fragment-omission fraction such as 10% for modified nucleosides) and a noise generator such as smiter.noise_functions.UniformNoiseInjector (e.g. 1000 counts of white noise). Set general parameters including gradient_length (e.g. 80 min for the nucleoside standard mix, 140 min for the Bruderer proteomics run) and scan time difference (60 ms). Run the simulation with smiter.synthetic_mzml.write_mzml, passing peak_properties, fragmentor, and noise injector, to write the synthetic run to an mzML file modelled on the complex_mix_simulation.py example. Isotope patterns are computed with pyQms and peak shapes use gauss-, gamma-, or exponentially-modified gaussian distributions; mass accuracy can be perturbed via MACHINE_OFFSET_IN_PPM (e.g. 0 vs 4 ppm) to test quantification sensitivity.

## Related tools

- **SMITER** (Core library whose smiter.synthetic_mzml.write_mzml function runs the simulation and writes the synthetic mzML; fragmentors and noise injectors are supplied as modular components) — https://github.com/LeidelLab/SMITER
- **Python** (Language in which the simulation script (e.g. complex_mix_simulation.py) is written and executed; SMITER is installed via pip or setup.py) — https://pypi.python.org/pypi/smiter

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('nucleoside_standards.csv'); from smiter.synthetic_mzml import write_mzml
```

## Evaluation signals

- The output mzML file loads in standard mzML parsers and its TIC can be extracted for downstream comparison
- The simulated run reproduces the expected injected features: peaks at the specified Modomics retention times with the specified isotopic patterns and the injected white noise (e.g. 1000 counts UniformNoiseInjector)
- Validation against published figures: TIC of the 80-min nucleoside simulation shows lower noise than the real 24-standard measurement (Figure 3); the 140-min proteomics simulation shows a constantly rising TIC without major intensity drops (Figure 4)
- ppm-offset control: runs at MACHINE_OFFSET_IN_PPM = 0 vs 4 ppm yield isotope-pattern match scores that degrade visibly at 4 ppm (Figure 5A/B)
- Installation sanity check via 'tox' passes, confirming the package and mzML writing pipeline are functional

## Limitations

- The default UniformNoiseInjector produces runs with lower noise than real measurements and omits unmodelled contaminants (e.g. the unidentified ~15 min peak in the real nucleoside run), so simulated TICs do not fully match instrument backgrounds
- Simulated proteomics TICs rise constantly over the gradient without the intensity drops and fluctuations of real measurements
- SMITER requires Python 3.7+ (LeidelLab README; 3.6+ in the MKoesters README) and the developers recommend using a virtual environment; the article's discussion section content (limitations prose) is not available

## Evidence

- [abstract] SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs.: "SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs."
- [methods] workflow step list: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [methods] noise injector: "Choose a Noise generator (e.g. smiter.noise_functions.UniformNoiseInjector)"
- [readme] readme features: "* usage of highly-accurate isotopic patterns enabled by `pyQms`_
* feature scaling by gauss-, gamma- and exponentially-modified gaussian distributions"
- [readme] readme testing: "To test the package and correct installation::

    user@localhost:~/smiter$ tox"
