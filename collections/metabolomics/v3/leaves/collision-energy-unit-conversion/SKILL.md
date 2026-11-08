---
name: collision-energy-unit-conversion
description: Use when merging MS/MS library entries (e.g., .msp/.mgf from NIST'17,
  MS-DIAL, MSnLib) that annotate collision energies in mixed formats such as NCE and
  eV, and a uniform CE scale is needed before filtering or training.
when_to_use_negative:
- Spectra already annotated with collision energies in eV only — conversion would
  be redundant
- Datasets where collision energy metadata is absent entirely — entries must be excluded
  rather than converted
- Libraries not restricted to [M+H]+/[M-H]- (or [M]+/[M]-) precursor types that fail
  the paper's filtering criteria
tools:
- name: FIORA
  role: Spectral prediction framework whose library curation pipeline performs the
    NCE-to-eV conversion during data preparation; downstream fiora-predict/fiora-train
    consume CE values from the 'CE' column of CSV inputs.
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:eb7c0110398b4a6872b741b947efe3a85d50c0a691af25932a200b5c03b657b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# collision-energy-unit-conversion

## Summary

Normalizes collision energy metadata across heterogeneous spectral libraries (NIST'17, MS-DIAL, MSnLib) by converting nominal collision energies (NCE) to electron volts (eV), so that all retained MS/MS spectra can be curated and modeled under a single, comparable CE scale.

## When to use

Use when merging MS/MS library entries (e.g., .msp/.mgf from NIST'17, MS-DIAL, MSnLib) that annotate collision energies in mixed formats such as NCE and eV, and a uniform CE scale is needed before filtering or training.

## When NOT to use

- Spectra already annotated with collision energies in eV only — conversion would be redundant
- Datasets where collision energy metadata is absent entirely — entries must be excluded rather than converted
- Libraries not restricted to [M+H]+/[M-H]- (or [M]+/[M]-) precursor types that fail the paper's filtering criteria

## Inputs

- Raw NIST'17 MS/MS library entries (spectral metadata with CE annotations)
- MS-DIAL spectral library in .msp format
- MSnLib raw MS2 spectral files (*_ms2.mgf pattern from the Zenodo record)
- CE-annotated precursor/pump-probe metadata parsed from the library files

## Outputs

- Curated library entries with collision energies uniformly expressed in eV
- Filtered spectral table (library.csv) with CE <= 100 eV, weight <= 1000 Da, and valid ionization modes

## How to apply

Parse raw NIST'17, MS-DIAL, and MSnLib library entries and convert nominal collision energies (NCE) to eV for every entry. Exclude any MS/MS spectra with missing collision energies or unclearly formatted CE fields, since these cannot be reliably mapped onto the eV scale. Enforce the resulting dataset's hard ceiling of maximum collision energy 100 eV together with the maximum compound weight of 1000 Da and the allowed ionization types ([M+H]+, [M-H]-, plus [M]+/[M]- for MSnLib). The rationale is that FIORA's training (e.g., on MSnLib HCD spectra) and prediction inputs both expect CE in a consistent numeric column, so unnormalized NCE values would break comparability across libraries and collision energies.

## Related tools

- **FIORA** (Spectral prediction framework whose library curation pipeline performs the NCE-to-eV conversion during data preparation; downstream fiora-predict/fiora-train consume CE values from the 'CE' column of CSV inputs.) — https://github.com/BAMeScience/fiora

## Examples

```
python resources/data/msnlib/download_msnlib.py && python resources/data/msnlib/preprocess_msnlib.py
```

## Evaluation signals

- No retained spectrum has a missing or unparseable collision energy field
- All collision energy values in the curated library.csv are in eV and fall at or below the 100 eV hard limit
- Entries converted from NCE can be cross-checked against their original NCE annotation by round-trip comparison
- The output library.csv is loadable as a table and contains only entries passing the 1000 Da / 100 eV / [M+H]+/[M-H]- filters
- For MSnLib preprocessing, the retained spectra also satisfy >=2 peak matches, >=50% intensity coverage, and <=90% precursor intensity

## Limitations

- MS/MS spectra with missing collision energies or unclear formats cannot be converted and must be excluded
- NIST'17 is a commercial library requiring a license, limiting public reproduction of this conversion step on that source
- CE metadata availability was a major constraint: FIORA was trained on only ~10,000 unique compounds, largely due to lack of experimental metadata, especially collision energies

## Evidence

- [full_text] Parse the raw NIST'17, MS-DIAL, and MSnLib library entries, convert nominal collision energies (NCE) to eV, and exclude any MS/MS spectra with missing or unclearly formatted collision energies.: "MS/MS spectra with missing collision energies or unclear formats were excluded."
- [discussion] The hard CE ceiling is applied alongside weight and ionization filters during curation.: "These requirements included a maximum weight of 1000 Da and maximum collision energy of 100 eV."
- [readme] The FIORA prediction/training input schema carries CE as an explicit per-query column, requiring consistent units.: "With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [readme] MSnLib preprocessing scripts used in the curation pipeline are bundled with the repository.: "Scripts for downloading and preprocessing MSnLib are provided in `resources/data/msnlib` (`download_msnlib.py` and `preprocess_msnlib.py`)."
