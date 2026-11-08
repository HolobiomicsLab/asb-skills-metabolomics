---
name: compound-input-csv-preparation
description: Use when a batch of compounds must be formatted for FIORA spectral prediction
  (e.g. test compound lists for benchmarking against CFM-ID/ICEBERG, or augmenting
  spectral libraries) and an fiora-predict-compatible CSV is needed.
when_to_use_negative:
- Compounds with inconsistencies between SMILES, MOL format and InChiKeys — these
  are filtered out by FIORA's preprocessing and should be cleaned before inclusion
- Adducts outside the ionization types FIORA supports, e.g. multi-charge or exotic
  precursor types beyond [M+H]+, [M-H]- ([M]+/[M]- only for MSnLib-derived use)
- Spectra with missing collision energies or unclear formats, which were excluded
  in the paper's pipeline
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2254
tools:
- name: FIORA
  role: Target tool whose CLI (fiora-predict) consumes the prepared CSV and produces
    msp/mgf spectra predictions
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Benchmark predictor run on the same compound list for runtime/quality comparison;
    input set must be shared across tools
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Benchmark predictor run via its Docker container (wishartlab/cfmid) over the
    same prepared compound list
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:01c7101dd75434becfd221c65d2f3bd859ac79847b88e7334a6f7f5e714bff8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - Docker (wishartlab/cfmid)
  - FIORA
  - ICEBERG
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2254
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# compound-input-csv-preparation

## Summary

Prepare a CSV query file with the exact header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' required by FIORA's fiora-predict CLI, so that tandem mass spectra (MS/MS), retention time (RT), and collision cross section (CCS) predictions can be run batch-wise on GPU or CPU.

## When to use

Use when a batch of compounds must be formatted for FIORA spectral prediction (e.g. test compound lists for benchmarking against CFM-ID/ICEBERG, or augmenting spectral libraries) and an fiora-predict-compatible CSV is needed.

## When NOT to use

- Compounds with inconsistencies between SMILES, MOL format and InChiKeys — these are filtered out by FIORA's preprocessing and should be cleaned before inclusion
- Adducts outside the ionization types FIORA supports, e.g. multi-charge or exotic precursor types beyond [M+H]+, [M-H]- ([M]+/[M]- only for MSnLib-derived use)
- Spectra with missing collision energies or unclear formats, which were excluded in the paper's pipeline

## Inputs

- Compound list with Name, SMILES, Precursor_type, CE, and Instrument_type per compound (CSV)
- Target adduct (Precursor_type), typically [M+H]+ or [M-H]-
- Collision energy (CE) and Instrument_type (e.g. HCD) matching the desired prediction conditions
- Packaged default FIORA model checkpoint under fiora/resources/models

## Outputs

- fiora-predict-compatible CSV input file (header: Name, SMILES, Precursor_type, CE, Instrument_type)
- Validated example output confirmation (mgf) matching examples/expected_output.mgf

## How to apply

Build a CSV whose header defines exactly the columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type', with one row per query compound; see the packaged examples/example_input.csv for a template. Set Precursor_type to the adduct of interest, most commonly [M+H]+ or [M-H]- (FIORA training also allowed [M]+/[M]- for MSnLib), and use the collision energy and instrument type of the target experiment (e.g. HCD). Load the packaged default model checkpoint under fiora/resources/models rather than supplying --model, since the CLI selects it automatically. Before committing to a full batch, run a small test and verify the output against examples/expected_output.mgf, or run pytest -v tests, to confirm the input is parsed correctly. Then run the full list with fiora-predict, optionally adding --dev cuda:0 for GPU acceleration, writing output in msp or mgf format.

## Related tools

- **FIORA** (Target tool whose CLI (fiora-predict) consumes the prepared CSV and produces msp/mgf spectra predictions) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Benchmark predictor run on the same compound list for runtime/quality comparison; input set must be shared across tools) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Benchmark predictor run via its Docker container (wishartlab/cfmid) over the same prepared compound list)

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --dev cuda:0
```

## Evaluation signals

- CSV header matches exactly: Name, SMILES, Precursor_type, CE, Instrument_type (case and order as in examples/example_input.csv)
- Small test batch output (mgf) matches the packaged examples/expected_output.mgf reference, or pytest -v tests passes
- Every row has a valid SMILES and a supported Precursor_type ([M+H]+ or [M-H]-); no rows with missing/unclear collision energies
- Full batch completes on GPU (e.g. ~10,000 spectra within about five minutes; ~0.03 s per prediction for 14,100 spectra) without parsing errors
- Same CSV feeds all tools (FIORA, CFM-ID, ICEBERG) in the benchmark so per-spectrum timing comparisons are on identical compound sets

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, so compound coverage and metadata (especially collision energies) in the input affect prediction quality
- The paper notes CFM-ID was trained on METLIN17 and FIORA/ICEBERG train/test separation had to be enforced by removing test compounds from training data — the input list's provenance matters for benchmark validity
- Single-step fragmentation only: compounds requiring multi-step fragmentation pathways are a known FIORA limitation

## Evidence

- [readme] Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type" and rows listing individual queries.: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [readme] By default, an open-source model is selected automatically, and predictions typically complete within a few seconds. For faster performance, specify a GPU device using the --dev option.: "By default, an open-source model is selected automatically, and predictions typically complete within a few seconds. For faster performance, specify a GPU device using the `--dev` option"
- [discussion] we filtered for ionization types [M+H]+ and [M-H]- (in case of MSnLib [M]+ and [M]- ions were considered as well): "we filtered for ionization types [M+H]+ and [M-H]- (in case of MSnLib [M]+ and [M]- ions were considered as well)"
- [discussion] MS/MS spectra with missing collision energies or unclear formats were excluded.: "MS/MS spectra with missing collision energies or unclear formats were excluded."
