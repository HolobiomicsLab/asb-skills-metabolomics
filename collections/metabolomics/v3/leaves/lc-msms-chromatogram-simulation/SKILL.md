---
name: lc-msms-chromatogram-simulation
description: Use when the goal is to generate a gold-standard synthetic mzML for LC-MS/MS
  (peptides, nucleosides, or lipids defined by chemical formulas) and benchmark it
  against a real run, e.g. reproducing a 140-min proteomics gradient simulation.
when_to_use_negative:
- 'The input is not representable by chemical formulas or requires a fragmentation/noise
  model not implemented in SMITER (default: peptide, two nucleoside, and one lipid
  fragmentation models).'
- The task requires realistic fluctuating TIC behavior with major intensity drops
  matching a real measurement — the SMITER-simulated TIC rises constantly without
  such drops.
- The goal is processing an existing raw mzML rather than generating a synthetic one.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMITER
  role: Python library used to construct peak properties from csv, apply the PeptideFragmentor
    and UniformNoiseInjector, and write the synthetic mzML
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: runtime environment for running the SMITER simulation script (e.g. complex_mix_simulation.py)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:5b12650008bdc542fd4c26aef6c9ddeb31179a9195388433c0b26a73d6ce4cb6
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
yes_rate: 0.7
n_claims: 10
---

# lc-msms-chromatogram-simulation

## Summary

Simulate a synthetic LC-MS/MS run (mzML) with SMITER from a list of molecules defined by chemical formulas, then compare the simulated total ion chromatogram (TIC) against a real measurement to identify differences such as steadily rising TIC and absence of intensity drops.

## When to use

Use when the goal is to generate a gold-standard synthetic mzML for LC-MS/MS (peptides, nucleosides, or lipids defined by chemical formulas) and benchmark it against a real run, e.g. reproducing a 140-min proteomics gradient simulation.

## When NOT to use

- The input is not representable by chemical formulas or requires a fragmentation/noise model not implemented in SMITER (default: peptide, two nucleoside, and one lipid fragmentation models).
- The task requires realistic fluctuating TIC behavior with major intensity drops matching a real measurement — the SMITER-simulated TIC rises constantly without such drops.
- The goal is processing an existing raw mzML rather than generating a synthetic one.

## Inputs

- csv file of molecules (e.g. identified peptides from the Bruderer et al. proteomics dataset) convertible via smiter.lib.csv_to_peak_properties
- peak properties dict with molecule formulas and chromatographic peak properties
- simulation parameters (e.g. gradient_length = 140 min, scan time difference = 60 ms)

## Outputs

- synthetic mzML file (e.g. complex_mix_simulation.mzml)
- TIC comparison of simulated vs. real LC-MS/MS run (Figure 4-style comparison)

## How to apply

First load the identified molecule list (e.g. peptides identified in the Bruderer et al. dataset) and build the peak properties dict with smiter.lib.csv_to_peak_properties from a csv file as in example_data. Select smiter.fragmentation_functions.PeptideFragmentor as the MS2 fragmentor and smiter.noise_functions.UniformNoiseInjector as the noise injector. Define general parameters such as gradient_length (140 min for the Bruderer proteomics case) and the scan time difference (60 ms). Run the simulation and write the output file with smiter.synthetic_mzml.write_mzml, passing peak_properties, the noise injector, and the fragmentor as parameters, saving e.g. complex_mix_simulation.mzml. Judge correctness by comparing the simulated TIC to the real measurement: the simulated run should show more distinct features and a constantly rising TIC over the long RT window without major intensity drops, unlike the fluctuating real measurement.

## Related tools

- **SMITER** (Python library used to construct peak properties from csv, apply the PeptideFragmentor and UniformNoiseInjector, and write the synthetic mzML) — https://github.com/LeidelLab/SMITER
- **Python** (runtime environment for running the SMITER simulation script (e.g. complex_mix_simulation.py))

## Examples

```
import smiter; peak_properties = smiter.lib.csv_to_peak_properties('peptides.csv'); smiter.synthetic_mzml.write_mzml(peak_properties, fragmentor=smiter.fragmentation_functions.PeptideFragmentor(), noise_injector=smiter.noise_functions.UniformNoiseInjector(), gradient_length=140)
```

## Evaluation signals

- Output file is a valid mzML produced by smiter.synthetic_mzml.write_mzml containing MS1 and MS2 scans for the specified peptides.
- The simulated TIC rises constantly over the long RT window with no major intensity drops, matching the reported Figure 4 behavior.
- The simulated run shows more distinct features than the real Bruderer measurement in the comparison.
- Simulated gradient spans the full gradient_length of 140 min with scans spaced by the 60 ms scan time difference.
- Installation sanity check passes via `tox` as recommended by the README.

## Limitations

- The default simulated TIC rises constantly with no major intensity drops, in contrast to real measurements which fluctuate, limiting realism of the chromatogram envelope.
- Default fragmentation coverage is limited to several peptide methods, two nucleoside models, and one lipid model; other models must be implemented by the user due to the modular design.
- No changelog or discussion-section limitations were available for the tool.

## Evidence

- [methods] workflow steps from methods: "create the peak properties dict (You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`)"
- [methods] write step: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [results] TIC comparison finding: [evidence span withheld — anchor status: not_found]
- [abstract] modular design claim: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [readme] installation from README: "To test the package and correct installation::

    user@localhost:~/smiter$ tox"
