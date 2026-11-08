---
name: peak-property-table-preparation
description: Use when building a SMITER simulation and you need to turn a list of
  identified peptides with retention times and intensities (e.g. from a database search
  at 1% FDR) into the peak-properties csv/dict format SMITER expects.
when_to_use_negative:
- Input is already a SMITER peak-properties dictionary — go straight to choosing a
  fragmentor and noise generator and calling write_mzml
- You need to re-search or re-validate raw spectra (FDR filtering, Percolator scoring)
  — that is upstream identification, not peak-properties preparation
- Input is raw instrument data rather than identified peptides with retention times
  — SMITER builds mzML from formulas, not from acquired files
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMITER
  role: provides smiter.lib.csv_to_peak_properties, which converts the csv file (as
    in example_data) into the peak-properties dict used for simulation
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: runtime environment in which the csv-to-peak-properties conversion and downstream
    write_mzml simulation are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:f4b01320d4db0130d1aaece903a2068d021ee502561878d63b1e1a7252b47fb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# peak-property-table-preparation

## Summary

Prepare the csv-encoded peak-properties table (peptide sequences, intensities, retention times) that SMITER consumes to drive a synthetic LC-MS/MS mzML simulation, converting externally derived identifications into a SMITER-ready dictionary via smiter.lib.csv_to_peak_properties.

## When to use

Use when building a SMITER simulation and you need to turn a list of identified peptides with retention times and intensities (e.g. from a database search at 1% FDR) into the peak-properties csv/dict format SMITER expects.

## When NOT to use

- Input is already a SMITER peak-properties dictionary — go straight to choosing a fragmentor and noise generator and calling write_mzml
- You need to re-search or re-validate raw spectra (FDR filtering, Percolator scoring) — that is upstream identification, not peak-properties preparation
- Input is raw instrument data rather than identified peptides with retention times — SMITER builds mzML from formulas, not from acquired files

## Inputs

- csv file of peptide identifications with retention times and intensities, in the same layout as SMITER's example_data files
- PSM list filtered to 1% FDR (e.g. from X!Tandem/ursgal search of the Bruderer et al. reference run validated with Percolator v3.4.0)

## Outputs

- peak_properties dictionary (peak_properties.csv equivalent) usable as input to smiter.synthetic_mzml.write_mzml
- synthetic mzML file of the simulated LC-MS/MS run

## How to apply

First obtain peptide identifications and retention times from an external search — e.g. searching the Bruderer et al. reference run with X!Tandem via ursgal at 5 ppm precursor and 0.02 Da fragment tolerance, validating PSMs with Percolator v3.4.0 and filtering to 1% FDR. Write these identifications as a csv file laid out the same way as the example_data files shipped with SMITER. Load the csv into a SMITER peak-properties dictionary with smiter.lib.csv_to_peak_properties, as described in the paper's step 1 of the simulation workflow. The resulting dict is then passed, together with a fragmentor (e.g. fragmentation_functions.PeptideFragmentor), a noise generator (e.g. smiter.noise_functions.UniformNoiseInjector), and general parameters such as gradient_length, to smiter.synthetic_mzml.write_mzml to produce the simulated mzML. The rationale is that SMITER simulates any biomolecule from chemical formulas, so the peak-properties table must supply sequences/intensities/RTs from which formulas and elution profiles are derived.

## Related tools

- **SMITER** (provides smiter.lib.csv_to_peak_properties, which converts the csv file (as in example_data) into the peak-properties dict used for simulation) — https://github.com/LeidelLab/SMITER
- **Python** (runtime environment in which the csv-to-peak-properties conversion and downstream write_mzml simulation are executed)

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('peak_properties.csv')
```

## Evaluation signals

- peak_properties.csv exists and parses as a valid csv
- peptide entries in the peak-properties table correspond to the 1% FDR PSM set from the Percolator-validated search
- csv layout matches SMITER's example_data files
- csv_to_peak_properties returns a dict that is accepted by smiter.synthetic_mzml.write_mzml without parameter errors
- resulting mzML contains the expected peptide features (isotopic patterns via pyQms) at the specified retention times and intensities

## Limitations

- SMITER requires Python 3.7 or higher (LeidelLab README; the MKoesters README states 3.6 or higher)
- No changelog or discussion-section prose is available, so limitations on csv schema flexibility (e.g. additional columns) are undocumented
- Retention times and intensities must be supplied externally — SMITER itself does not predict them by default (RT prediction must be implemented via the Python ecosystem)

## Evidence

- [methods] create the peak properties dict from a csv file using smiter.lib.csv_to_peak_properties: "create the peak properties dict (You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`)"
- [methods] run the simulation and write the resulting mzML using smiter.synthetic_mzml.write_mzml: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [other] csv layout matching example_data: [evidence span withheld — anchor status: not_found]
- [readme] SMITER abstracts molecules to chemical formulas: "Python library to create synthetic mzMLs file based on chemical formulas. All molecules can be simulated du to abstraction to chemical formulas."
- [other] 1% FDR identification workflow feeding the peak properties: [evidence span withheld — anchor status: not_found]
