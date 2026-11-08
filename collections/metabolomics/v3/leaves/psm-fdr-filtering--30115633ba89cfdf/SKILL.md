---
name: psm-fdr-filtering
description: Use when raw LC-MS/MS spectra (e.g. an mzML from a PRIDE/ProteomeXchange
  deposit) must be converted into a confident, FDR-controlled peptide identification
  list with retention times.
when_to_use_negative:
- The input is already a validated peptide list or feature table at the desired FDR
  — re-searching the spectra is unnecessary
- The downstream tool expects chemical formulas or peak properties, not identifications
  — a search/FDR step alone will not produce the required peak-properties dictionary
  without an additional conversion (e.g. smiter.lib.csv_to_peak_properties)
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: X!Tandem
  role: Search engine used through the ursgal framework to identify PSMs with 5 ppm
    precursor and 0.02 Da fragment tolerance
- name: Percolator
  role: PSM validation and re-scoring tool (v3.4.0) used to filter identifications
    to 1% FDR
- name: ursgal
  role: Python framework that wraps and drives the X!Tandem search
- name: SMITER
  role: 'Downstream consumer: the 1% FDR peptide list csv is converted to a peak-properties
    dictionary via smiter.lib.csv_to_peak_properties to simulate the LC-MS/MS run'
  repo: https://github.com/LeidelLab/SMITER
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:61cdef5b31959ec5b74cedf5b818b04a85a66accbd8344ac3c7f7500e0f856b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Percolator
  - Python
  - SMITER
  - X!Tandem
  - ursgal
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# psm-fdr-filtering

## Summary

Search LC-MS/MS spectra for peptide-spectrum matches (PSMs) and filter them to a target false discovery rate, yielding a confident peptide list with retention times that can seed downstream analyses such as simulated mzML runs.

## When to use

Use when raw LC-MS/MS spectra (e.g. an mzML from a PRIDE/ProteomeXchange deposit) must be converted into a confident, FDR-controlled peptide identification list with retention times.

## When NOT to use

- The input is already a validated peptide list or feature table at the desired FDR — re-searching the spectra is unnecessary
- The downstream tool expects chemical formulas or peak properties, not identifications — a search/FDR step alone will not produce the required peak-properties dictionary without an additional conversion (e.g. smiter.lib.csv_to_peak_properties)

## Inputs

- Raw LC-MS/MS spectra file from a public deposit (e.g. mzML of the Bruderer et al. reference run from PRIDE/ProteomeXchange)
- Protein sequence database for the target search engine (X!Tandem via ursgal)
- Search parameter set: 5 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed carbamidomethylation, optional methionine oxidation and N-terminal acetylation

## Outputs

- Peptide list (identifications plus retention times) filtered to 1% FDR
- csv file in the same layout as the example_data files (e.g. peak_properties.csv consumable by smiter.lib.csv_to_peak_properties)

## How to apply

Retrieve the reference LC-MS/MS run from its public PRIDE/ProteomeXchange deposit and search the spectra with X!Tandem through the ursgal framework using 5 ppm precursor tolerance and 0.02 Da fragment tolerance, with fixed carbamidomethylation and optional methionine oxidation and N-terminal acetylation. Validate the resulting PSMs with Percolator v3.4.0 and filter to 1% FDR to obtain the peptide list. Export the peptide identifications and corresponding retention times as a csv file laid out like the example_data files of the downstream tool. Finally, verify that the csv parses correctly and that its peptide entries correspond to the 1% FDR PSM set before using it (e.g. converting it via smiter.lib.csv_to_peak_properties into a SMITER peak-properties dictionary).

## Related tools

- **X!Tandem** (Search engine used through the ursgal framework to identify PSMs with 5 ppm precursor and 0.02 Da fragment tolerance)
- **Percolator** (PSM validation and re-scoring tool (v3.4.0) used to filter identifications to 1% FDR)
- **ursgal** (Python framework that wraps and drives the X!Tandem search)
- **SMITER** (Downstream consumer: the 1% FDR peptide list csv is converted to a peak-properties dictionary via smiter.lib.csv_to_peak_properties to simulate the LC-MS/MS run) — https://github.com/LeidelLab/SMITER

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('peak_properties.csv')
```

## Evaluation signals

- peak_properties.csv exists and parses as valid csv
- Peptide entries in the csv correspond exactly to the 1% FDR PSM set from the Percolator-validated search
- Search parameters applied match the spec: 5 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed carbamidomethylation, optional Met oxidation and N-terminal acetylation
- Csv layout matches the example_data files of the downstream tool (SMITER)

## Limitations

- FDR control is applied at the PSM level via Percolator; peptide- or protein-level FDR is not explicitly addressed in this workflow
- The csv conversion step (smiter.lib.csv_to_peak_properties) is only demonstrated for the example_data layout, so deviating column layouts may break the conversion

## Evidence

- [other] Peptide identifications for the SMITER proteomics simulation were obtained by searching the Bruderer et al. reference run with X!Tandem via the ursgal framework, using 5 ppm precursor and 0.02 Da fragment tolerance.: [evidence span withheld — anchor status: not_found]
- [other] Validate the PSMs with Percolator v3.4.0 and filter to 1% FDR to obtain the peptide list.: [evidence span withheld — anchor status: not_found]
- [other] Write the peptide list as a csv file in the same layout as the example_data files.: [evidence span withheld — anchor status: not_found]
- [methods] You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
