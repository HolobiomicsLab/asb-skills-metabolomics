---
name: cross-tool-runtime-comparison
description: Use when you need to quantify and compare prediction speed of competing
  MS/MS spectral prediction tools on the same compound set, including the effect of
  GPU vs CPU execution.
when_to_use_negative:
- The goal is to compare prediction quality (e.g., spectral similarity or peak intensity
  coverage) rather than runtime — use the benchmark-quality skill instead
- You only need single-spectrum latency for one tool; a cross-tool batch benchmark
  on the full test set is unnecessary
- Retraining CFM-ID on your own dataset — the article found this computationally infeasible;
  comparison uses CFM-ID as trained on METLIN17
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Spectral prediction tool being benchmarked; run via fiora-predict on GPU (--dev
    cuda:0) and CPU to measure device-dependent runtime
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: State-of-the-art fragmentation baseline whose runtime is measured over the
    same test compounds using its Docker container
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: 'Spectral prediction baseline run over the same test compounds following its
    ms-pred repository setup; supports CPU-only inference via cuda_devices: None'
  repo: https://github.com/coleygroup/ms-pred
- name: Docker (wishartlab/cfmid)
  role: Container runtime used to execute CFM-ID spectral prediction for the timing
    benchmark
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:409f8902d33c45784781a5810abb7d18db99ec74b106f877f6a3103cce71548d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-tool-runtime-comparison

## Summary

Benchmark wall-clock runtime of FIORA spectral prediction against CFM-ID and ICEBERG on an identical test compound set, comparing FIORA on GPU (cuda:0) versus CPU and computing per-spectrum prediction times for each tool/device combination (Table 2 of the FIORA paper).

## When to use

Use when you need to quantify and compare prediction speed of competing MS/MS spectral prediction tools on the same compound set, including the effect of GPU vs CPU execution.

## When NOT to use

- The goal is to compare prediction quality (e.g., spectral similarity or peak intensity coverage) rather than runtime — use the benchmark-quality skill instead
- You only need single-spectrum latency for one tool; a cross-tool batch benchmark on the full test set is unnecessary
- Retraining CFM-ID on your own dataset — the article found this computationally infeasible; comparison uses CFM-ID as trained on METLIN17

## Inputs

- Test compound list as CSV with header columns: Name, SMILES, Precursor_type, CE, Instrument_type
- Packaged default FIORA model checkpoint (fiora/resources/models)
- CFM-ID Docker image (wishartlab/cfmid)
- ICEBERG setup from the coleygroup/ms-pred repository

## Outputs

- prediction_time_benchmark.csv with total and per-spectrum prediction times for FIORA GPU, FIORA CPU, CFM-ID, and ICEBERG
- Predicted spectra files (mgf/msp) from fiora-predict used to confirm batch completion counts

## How to apply

Prepare the benchmark compound list as a CSV with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' and load FIORA's packaged default model from fiora/resources/models. Run fiora-predict over the full compound list twice — once with --dev cuda:0 (GPU) and once on CPU — recording total wall-clock time per batch. Run CFM-ID over the identical compounds via the wishartlab/cfmid Docker container, and ICEBERG via its ms-pred repository setup (CPU-only inference is feasible by setting cuda_devices to None), recording total runtime for each. Divide each total time by the number of test spectra to get per-spectrum times, then tabulate total and per-spectrum times for FIORA GPU, FIORA CPU, CFM-ID, and ICEBERG into a single comparison CSV. The rationale is that runtime, like prediction quality, is a differentiator: FIORA's GPU-accelerated graph neural network processes spectra in bulk, while CFM-ID requires containerized execution and ICEBERG follows its own repository pipeline.

## Related tools

- **FIORA** (Spectral prediction tool being benchmarked; run via fiora-predict on GPU (--dev cuda:0) and CPU to measure device-dependent runtime) — https://github.com/BAMeScience/fiora
- **CFM-ID** (State-of-the-art fragmentation baseline whose runtime is measured over the same test compounds using its Docker container) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (Spectral prediction baseline run over the same test compounds following its ms-pred repository setup; supports CPU-only inference via cuda_devices: None) — https://github.com/coleygroup/ms-pred
- **Docker (wishartlab/cfmid)** (Container runtime used to execute CFM-ID spectral prediction for the timing benchmark) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --dev cuda:0
```

## Evaluation signals

- FIORA GPU timing is on the order of the published reference: about 7 minutes for 14,100 spectra (~0.03 s per prediction)
- Every tool/device combination (FIORA GPU, FIORA CPU, CFM-ID, ICEBERG) has a complete total-time and per-spectrum entry in prediction_time_benchmark.csv
- FIORA GPU runtime is roughly 30x faster than FIORA CPU on the same compound set; FIORA CPU is about 4x faster than CFM-ID but slightly slower than ICEBERG
- Per-spectrum times in the table equal total time divided by the number of test spectra for each row
- All runs consumed the identical input CSV (same compound set, precursor types, collision energies) so timing comparisons are apples-to-apples

## Limitations

- Runtime benchmarks are hardware-dependent; GPU results require a CUDA device and CPU-only results depend on the host machine
- CFM-ID was trained on METLIN17 and retraining with the FIORA dataset was found computationally infeasible, so the comparison uses the shipped CFM-ID model
- ICEBERG pretrained weights may require a NIST license or use publicly available MassSpecGym weights, whose predictions differ; NIST'17 is a commercial library requiring a license

## Evidence

- [abstract] GPU acceleration enabling rapid prediction: "Utilizing GPU acceleration, FIORA enables rapid validation of putative compound annotations and large-scale expansion of spectral reference libraries"
- [readme] FIORA README GPU device option: "For faster performance, specify a GPU device using the `--dev` option (e.g., `--dev cuda:0`)."
- [discussion] CFM-ID Docker container reference: "running with a docker container provided on Docker Hub (https://hub.docker.com/r/wishartlab/cfmid)"
- [readme] ICEBERG CPU-only inference support: [evidence span withheld — anchor status: not_found]
- [intro] CFM-ID benchmarked against FIORA: "We benchmark the performance against the top-performing methods, CFM-ID and ICEBERG."
