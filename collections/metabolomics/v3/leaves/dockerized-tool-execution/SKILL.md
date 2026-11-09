---
name: dockerized-tool-execution
description: Use when a runtime benchmark must compare prediction tools with heterogeneous
  installs (CFM-ID via Docker, ICEBERG via its repo, FIORA natively) over an identical
  compound list, and per-spectrum prediction times must be tabulated.
when_to_use_negative:
- When the goal is prediction quality (spectral similarity) rather than runtime —
  the article's benchmarking of prediction quality uses separate CASMI 2016/2022 workflows
  with collision-energy matching.
- When retraining CFM-ID is required — the authors found 'Retraining CFM-ID with our
  dataset was found computationally infeasible', so Dockerized CFM-ID is only usable
  for prediction as-is.
- When compounds require multi-step fragmentation — FIORA lacks multi-step fragmentation,
  so a runtime comparison of single-step prediction does not cover that use case.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID
  role: State-of-the-art fragmentation predictor executed via its Docker container
    (wishartlab/cfmid) during the runtime benchmark
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: Fragmentation predictor run from its repository (ms-pred) over the same test
    compounds to record runtime
  repo: https://github.com/samgoldman97/ms-pred
- name: FIORA
  role: Graph neural network spectral predictor timed both on GPU (--dev cuda:0) and
    CPU as the reference runtime point
  repo: https://github.com/BAMeScience/fiora
- name: Docker
  role: Container runtime used to execute CFM-ID without a native installation
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:c4e0d83bd58a36e16cbbfa1af218dad37c3ce525217cb4b004cff8f0d9899ae9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - Docker
  - Docker (wishartlab/cfmid)
  - FIORA
  - ICEBERG
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dockerized-tool-execution

## Summary

Benchmark external fragmentation-prediction tools by running them inside their published Docker containers (e.g., the wishartlab/cfmid Docker image for CFM-ID) alongside native repositories like ICEBERG's ms-pred, recording wall-clock runtimes over the same test compound set to reproduce the FIORA runtime comparison (Table 2).

## When to use

Use when a runtime benchmark must compare prediction tools with heterogeneous installs (CFM-ID via Docker, ICEBERG via its repo, FIORA natively) over an identical compound list, and per-spectrum prediction times must be tabulated.

## When NOT to use

- When the goal is prediction quality (spectral similarity) rather than runtime — the article's benchmarking of prediction quality uses separate CASMI 2016/2022 workflows with collision-energy matching.
- When retraining CFM-ID is required — the authors found 'Retraining CFM-ID with our dataset was found computationally infeasible', so Dockerized CFM-ID is only usable for prediction as-is.
- When compounds require multi-step fragmentation — FIORA lacks multi-step fragmentation, so a runtime comparison of single-step prediction does not cover that use case.

## Inputs

- CSV of test compounds with header columns: Name, SMILES, Precursor_type, CE, Instrument_type
- Packaged default FIORA model checkpoint (fiora/resources/models)
- Docker image wishartlab/cfmid pulled from hub.docker.com/r/wishartlab/cfmid
- ICEBERG setup from coleygroup/ms-pred repository

## Outputs

- prediction_time_benchmark.csv with total and per-spectrum times for FIORA GPU, FIORA CPU, CFM-ID, and ICEBERG
- Predicted MS/MS spectra per tool (mgf/msp for FIORA)
- Recorded wall-clock runtime logs per tool/device combination

## How to apply

Prepare one shared test-compound CSV with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' so every tool consumes the same queries. For CFM-ID, execute spectral prediction inside the wishartlab/cfmid Docker container from Docker Hub rather than a local install, recording total wall-clock time over the full test list. Run ICEBERG from its repository setup (coleygroup/ms-pred) and FIORA with fiora-predict both on GPU (--dev cuda:0) and CPU, timing each batch. Divide each total time by the number of test spectra to get per-spectrum prediction times, then tabulate FIORA GPU, FIORA CPU, CFM-ID, and ICEBERG into a single comparison table (prediction_time_benchmark.csv). Note the article's finding that FIORA is ~30x faster on GPU than CPU, ~10,000 spectra in five minutes, and four times faster than CFM-ID on CPU while slightly slower than ICEBERG.

## Related tools

- **CFM-ID** (State-of-the-art fragmentation predictor executed via its Docker container (wishartlab/cfmid) during the runtime benchmark) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (Fragmentation predictor run from its repository (ms-pred) over the same test compounds to record runtime) — https://github.com/samgoldman97/ms-pred
- **FIORA** (Graph neural network spectral predictor timed both on GPU (--dev cuda:0) and CPU as the reference runtime point) — https://github.com/BAMeScience/fiora
- **Docker** (Container runtime used to execute CFM-ID without a native installation) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --dev cuda:0; docker run wishartlab/cfmid <cfmid_predict_command>
```

## Evaluation signals

- FIORA GPU timing should be on the order of the published reference: about 7 minutes for 14,100 spectra (~0.03 s per prediction).
- Every tool/device combination (FIORA GPU, FIORA CPU, CFM-ID, ICEBERG) must have a complete total-time and per-spectrum entry in the table.
- FIORA GPU runtime should be roughly 30x faster than FIORA CPU over the same batch.
- FIORA on CPU should land around four times faster than CFM-ID but slightly slower than ICEBERG.
- The same input CSV (Name, SMILES, Precursor_type, CE, Instrument_type) must have been consumed by all tools so per-spectrum times are comparable.

## Limitations

- Retraining CFM-ID with the authors' dataset was found computationally infeasible, so the Dockerized CFM-ID benchmark reflects the pre-trained METLIN17 model only, without guaranteed train/test compound separation.
- Dockerized-tool runtimes depend on the host hardware and container overhead; the article's GPU figures assume a CUDA device (cuda:0), and CPU-only inference will be substantially slower.
- FIORA itself was trained on a relatively small dataset of ~10,000 unique compounds, limiting scope of what the timed model represents.

## Evidence

- [discussion] CFM-ID Docker: "running with a docker container provided on Docker Hub (https://hub.docker.com/r/wishartlab/cfmid)"
- [other] CPU vs GPU runtime finding: [evidence span withheld — anchor status: not_found]
- [readme] input format: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [readme] GPU device flag: "For faster performance, specify a GPU device using the `--dev` option (e.g., `--dev cuda:0`)."
- [discussion] CFM-ID retraining infeasible: "Retraining CFM-ID with our dataset was found computationally infeasible."
