---
name: pipeline-wallclock-runtime-benchmarking
description: Use when you need to quantify the computational cost of a multi-step
  preprocessing pipeline on a defined sample set, e.g. to demonstrate that a 70-sample
  1H-NMR workflow completes on ordinary hardware.
when_to_use_negative:
- You need algorithmic complexity or scaling analysis across varying sample counts,
  not a single end-to-end wall-clock figure for one fixed dataset.
- The pipeline is already parallelized/distributed and you need per-step or per-core
  profiling rather than one total runtime number.
- Input data is not the raw FID/spectra stage — timing a downstream statistics step
  on an already preprocessed feature table measures a different cost.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3520
tools:
- name: Protomix
  role: Package whose full preprocessing chain (extract_params through normalization)
    is the workload being timed
  repo: https://github.com/mzniber/protomix
- name: Python
  role: Runtime environment providing the time module used to measure wall-clock elapsed
    time
- name: pandas
  role: Used to write the runtime measurement and the normalized spectra table to
    CSV files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:677fa8ee29eccd2e880df6db4f45090fe00bb6be914ca244ca852715e1c0bd2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyicoshift
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pipeline-wallclock-runtime-benchmarking

## Summary

Measures the end-to-end wall-clock execution time of a full preprocessing pipeline (here, Protomix 1H-NMR preprocessing of 70 urine spectra) by bracketing the workflow with a Python timer and reporting elapsed runtime alongside the processed output.

## When to use

Use when you need to quantify the computational cost of a multi-step preprocessing pipeline on a defined sample set, e.g. to demonstrate that a 70-sample 1H-NMR workflow completes on ordinary hardware.

## When NOT to use

- You need algorithmic complexity or scaling analysis across varying sample counts, not a single end-to-end wall-clock figure for one fixed dataset.
- The pipeline is already parallelized/distributed and you need per-step or per-core profiling rather than one total runtime number.
- Input data is not the raw FID/spectra stage — timing a downstream statistics step on an already preprocessed feature table measures a different cost.

## Inputs

- Bruker-formatted 1H-NMR spectra directory (NMRDataset.zip from Zenodo record 13467227, 70 urine samples)
- acqus_df acquisition-parameter table from Protomix extract_params
- fids_df Free Induction Decay table from Protomix extract_fids

## Outputs

- preprocessed_spectra_normalized.csv (final binned, TotalArea-normalized spectra table)
- preprocessing_runtime.csv (single-row table with elapsed wall-clock seconds)

## How to apply

Define the pipeline boundaries explicitly: start timing after data acquisition steps (extract_params, extract_fids from the Bruker spectra directory of the Zenodo 13467227 NMRDataset) and stop after the final normalized table is produced. Import the standard library time module, record time at pipeline start, run the full Protomix chain (group_delay_removal, solvent_residuals_removal, apodization with LB=5, zero_filling to target_points=10000, fourier_transform, phase_correction, internal_referencing, baseline_correction with airpls, region_removal, negative_values_zeroing, Icoshift peak alignment, window_selection, binning at bin_size=0.1, TotalArea normalization), then record the stop time and compute elapsed = stop - start. Write the elapsed value to a dedicated CSV (e.g. preprocessing_runtime.csv) alongside the scientific output (preprocessed_spectra_normalized.csv) so the runtime claim is reproducible. The reference result from the article is full preprocessing of the 70-sample prostate cancer/BPH urine dataset in under two minutes on an 8-core laptop; use that as a sanity ceiling when benchmarking comparable hardware and sample counts.

## Related tools

- **Protomix** (Package whose full preprocessing chain (extract_params through normalization) is the workload being timed) — https://github.com/mzniber/protomix
- **Python** (Runtime environment providing the time module used to measure wall-clock elapsed time)
- **pandas** (Used to write the runtime measurement and the normalized spectra table to CSV files)

## Examples

```
import time; t0=time.time(); # ...run px group_delay_removal -> normalization...; pd.DataFrame({'runtime_s':[time.time()-t0]}).to_csv('preprocessing_runtime.csv')
```

## Evaluation signals

- preprocessing_runtime.csv exists and contains a single positive elapsed-time value in seconds
- Elapsed time for the full 70-sample Protomix workflow is under 120 seconds on an 8-core laptop (article reference point)
- preprocessed_spectra_normalized.csv is produced with the expected binned, TotalArea-normalized spectra so the timed run also completed scientifically
- Timer start is recorded after extract_params/extract_fids and stop after normalization, so the measurement covers exactly the declared pipeline boundaries
- Re-running the benchmark yields runtime in the same order of magnitude (no missing steps silently shortening the timed window)

## Limitations

- Runtime is hardware-dependent; the under-two-minute figure was obtained on an 8-core laptop and is not a portable guarantee.
- The benchmark measures one end-to-end wall-clock figure; it does not attribute cost to individual steps such as Icoshift alignment or airpls baseline correction.
- pyicoshift's multi-threading support is described as experimental, which can affect timing stability during peak alignment.

## Evidence

- [results] The complete preprocessing of the 70-sample urine dataset was finished efficiently in under two minutes on an 8-core laptop.: [evidence span withheld — anchor status: not_found]
- [abstract] Protomix offers automated, efficient preprocessing steps: "Protomix offers a set of automated, efficient, and user-friendly signal-preprocessing steps, tailored to streamline and enhance the preprocessing phase in metabolomics studies."
- [methods] dataset download from Zenodo: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [readme] pyicoshift experimental threading caveat: [evidence span withheld — anchor status: external_doc]
