---
name: gpu-accelerated-inference-timing
description: Use when comparing inference speed of spectral prediction tools (FIORA,
  CFM-ID, ICEBERG) on the same compound test set, or when quantifying how GPU acceleration
  (cuda:0) changes batch prediction throughput for tens of thousands of spectra.
when_to_use_negative:
- The goal is prediction quality assessment (spectral similarity scores) rather than
  runtime benchmarking — use the paper's prediction-quality comparison instead
- Only a handful of spectra need prediction; timing benchmarks require a full test
  compound batch to yield stable per-spectrum times
- Retraining CFM-ID on the paper's dataset is required — the authors found this 'computationally
  infeasible'
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FIORA
  role: Spectral prediction tool whose GPU (cuda:0) and CPU batch runtimes are benchmarked
    against CFM-ID and ICEBERG
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: State-of-the-art fragmentation baseline; run over the same test compounds
    via Docker to record comparison runtime
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: State-of-the-art fragmentation baseline; run following its repository setup
    to record comparison runtime
  repo: https://github.com/samgoldman97/ms-pred
- name: Docker (wishartlab/cfmid)
  role: Container runtime used to execute CFM-ID spectral prediction in the benchmark
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:8cf1b798a928d804edfe5b252f8c161005c84ffff221b2687a21a10a1870fe31
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gpu-accelerated-inference-timing

## Summary

Benchmarks the wall-clock runtime of FIORA's MS/MS spectral prediction against CFM-ID and ICEBERG across CPU and GPU devices, quantifying per-spectrum prediction times to demonstrate the ~30x GPU speedup that enables large-scale library expansion.

## When to use

Use when comparing inference speed of spectral prediction tools (FIORA, CFM-ID, ICEBERG) on the same compound test set, or when quantifying how GPU acceleration (cuda:0) changes batch prediction throughput for tens of thousands of spectra.

## When NOT to use

- The goal is prediction quality assessment (spectral similarity scores) rather than runtime benchmarking — use the paper's prediction-quality comparison instead
- Only a handful of spectra need prediction; timing benchmarks require a full test compound batch to yield stable per-spectrum times
- Retraining CFM-ID on the paper's dataset is required — the authors found this 'computationally infeasible'

## Inputs

- Test compound CSV with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' (e.g. examples/example_input.csv format)
- Packaged default FIORA model checkpoint under fiora/resources/models
- CFM-ID Docker image wishartlab/cfmid
- ICEBERG setup from github.com/samgoldman97/ms-pred / coleygroup/ms-pred

## Outputs

- Predicted spectra files in msp or mgf format per tool/device combination
- Total wall-clock runtime per tool/device combination
- Per-spectrum prediction times (total time divided by number of test spectra)
- Combined timing comparison table saved as prediction_time_benchmark.csv

## How to apply

Prepare the test compound set as a CSV with the required header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type', and use the packaged default FIORA model checkpoint from fiora/resources/models. Run fiora-predict over the full compound list twice, once with --dev cuda:0 (GPU) and once on CPU, recording total wall-clock time for each batch; per the paper, FIORA runs ~30x faster on GPU, predicting ~10,000 spectra within five minutes and remaining ~4x faster than CFM-ID even on CPU. Run CFM-ID spectral prediction over the identical compounds via its Docker container (wishartlab/cfmid), and run ICEBERG following the coleygroup/ms-pred repository setup, recording total runtimes for each. Compute per-spectrum prediction times by dividing each total time by the number of test spectra, then tabulate FIORA GPU, FIORA CPU, CFM-ID, and ICEBERG into a single comparison table saved as prediction_time_benchmark.csv. Validate that FIORA GPU timing matches the published reference order of magnitude (~7 minutes for 14,100 spectra, ~0.03 s per prediction).

## Related tools

- **FIORA** (Spectral prediction tool whose GPU (cuda:0) and CPU batch runtimes are benchmarked against CFM-ID and ICEBERG) — https://github.com/BAMeScience/fiora
- **CFM-ID** (State-of-the-art fragmentation baseline; run over the same test compounds via Docker to record comparison runtime) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (State-of-the-art fragmentation baseline; run following its repository setup to record comparison runtime) — https://github.com/samgoldman97/ms-pred
- **Docker (wishartlab/cfmid)** (Container runtime used to execute CFM-ID spectral prediction in the benchmark) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --dev cuda:0
```

## Evaluation signals

- FIORA GPU timing reproduces the published reference order of magnitude: about 7 minutes for 14,100 spectra (~0.03 s per prediction)
- FIORA GPU throughput is roughly 30x its CPU throughput, and FIORA CPU remains ~4x faster than CFM-ID
- prediction_time_benchmark.csv contains a complete total-time and per-spectrum entry for every tool/device combination (FIORA GPU, FIORA CPU, CFM-ID, ICEBERG)
- Input CSV header matches the required columns exactly: 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'
- Per-spectrum times equal total time divided by the number of test spectra for each row of the table

## Limitations

- GPU speedup depends on hardware; the published figures assume a CUDA device (cuda:0), so CPU-only machines will not reproduce the ~10,000 spectra in five minutes figure
- Retraining CFM-ID with the authors' dataset was found computationally infeasible, and CFM-ID was trained on METLIN17, so its runtime comparison is on a model trained on different data
- The benchmark measures wall-clock prediction time only and does not assess prediction quality across tools

## Evidence

- [results] FIORA runs ~30 times faster on GPU than on CPU, predicting around 10,000 spectra within five minutes; on CPU it is four times faster than CFM-ID but slightly slower than ICEBERG.: [evidence span withheld — anchor status: not_found]
- [readme] Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type": "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [discussion] running with a docker container provided on Docker Hub (https://hub.docker.com/r/wishartlab/cfmid): "running with a docker container provided on Docker Hub (https://hub.docker.com/r/wishartlab/cfmid)"
- [readme] By default, an open-source model is selected automatically, and predictions typically complete within a few seconds. For faster performance, specify a GPU device using the `--dev` option (e.g., `--dev cuda:0`).: "For faster performance, specify a GPU device using the `--dev` option (e.g., `--dev cuda:0`)"
- [abstract] Utilizing GPU acceleration, FIORA enables rapid validation of putative compound annotations and large-scale expansion of spectral reference libraries: "Utilizing GPU acceleration, FIORA enables rapid validation of putative compound annotations and large-scale expansion of spectral reference libraries"
