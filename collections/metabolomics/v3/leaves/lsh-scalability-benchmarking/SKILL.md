---
name: lsh-scalability-benchmarking
description: Use when you need to quantify how an MS raw-data processing implementation
  (e.g. LSH classification of TimsTOF frames) scales with parallelism before committing
  it to large-scale pipelines.
when_to_use_negative:
- When you need accuracy/ROC evaluation of LSH classification against intensity thresholding
  — that is the separate roc/ workflow, not a thread-scaling sweep.
- When generating synthetic averagine isotope patterns — use the createSynthetics/
  module instead.
- 'When GPU acceleration behavior is the question: the paper''s sweep is a CPU thread-count
  study, where TensorFlow 2 GPU acceleration is not required.'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mzBucket
  role: Repository whose scalability-study scripts perform the thread-sweep benchmarking
    of the LSH implementation
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: Data-access layer used to load raw TimsTOF frames (built on the low-level
    C++ API of OpenTIMS) for the benchmark runs
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Provides the LSH algorithm whose execution time is measured across thread
    counts
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: OpenTIMS
  role: Low-level C++ API for TimsTOF raw-data access underlying the benchmarked pipeline
- name: Python
  role: Language of the pybind11-exposed packages in which the scalability sweep scripts
    run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:e62bf965a056b0365cc029def2a6967e5e67a2cd729b400e7ccb7b09c879581d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - C++
  - OpenTIMS
  - Python
  - TensorFlow 2
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lsh-scalability-benchmarking

## Summary

Benchmarks the wall-clock scalability of the mzBucket/pyproteolizard LSH signal-classification implementation for TimsTOF raw data by sweeping CPU thread counts from 1 to 32 and recording execution times per run.

## When to use

Use when you need to quantify how an MS raw-data processing implementation (e.g. LSH classification of TimsTOF frames) scales with parallelism before committing it to large-scale pipelines.

## When NOT to use

- When you need accuracy/ROC evaluation of LSH classification against intensity thresholding — that is the separate roc/ workflow, not a thread-scaling sweep.
- When generating synthetic averagine isotope patterns — use the createSynthetics/ module instead.
- When GPU acceleration behavior is the question: the paper's sweep is a CPU thread-count study, where TensorFlow 2 GPU acceleration is not required.

## Inputs

- Raw TimsTOF data set from Zenodo record 5036526 (Bruker .d folder), unzipped into the repository's data folder
- Randomly selected subset of 100 MS1 frames from the acquisition (loaded via PyTimsDataHandle)

## Outputs

- scalability_results.csv written to data/output_scalability_study (wall-clock time vs. thread count, 1–32 threads)
- Plotted wall-clock-time-versus-threads curve saved under the plots directory

## How to apply

Download the raw TimsTOF data set from Zenodo record 5036526 and unzip it into the data folder of the mzBucket repository so scripts run reproducibly. Load the raw data via the pyproteolizard-data/OpenTIMS data-access layer (PyTimsDataHandle over a Bruker .d folder), randomly select 100 MS1 frames from the acquisition, and run the LSH algorithm from pyproteolizard-algorithm on those frames repeatedly while sweeping the thread count from 1 up to 32 threads, recording wall-clock execution time for each thread count; GPU acceleration via TensorFlow 2 is not needed for this CPU sweep. Aggregate the timings into a wall-clock-time-versus-threads table written to data/output_scalability_study/scalability_results.csv, then plot the timing curve under the plots directory. The sweep is judged successful when the curve shows the reported behavior — an initial decrease in wall-clock time with additional threads followed by saturation, i.e. parallelization overhead cancelling further speed gains.

## Related tools

- **mzBucket** (Repository whose scalability-study scripts perform the thread-sweep benchmarking of the LSH implementation) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (Data-access layer used to load raw TimsTOF frames (built on the low-level C++ API of OpenTIMS) for the benchmark runs) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Provides the LSH algorithm whose execution time is measured across thread counts) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **OpenTIMS** (Low-level C++ API for TimsTOF raw-data access underlying the benchmarked pipeline)
- **Python** (Language of the pybind11-exposed packages in which the scalability sweep scripts run)

## Examples

```
python src/python/scalability/scalability.py  # sweep 1-32 threads on 100 MS1 frames; writes data/output_scalability_study/scalability_results.csv
```

## Evaluation signals

- The output CSV in data/output_scalability_study contains one wall-clock timing per thread count for the 1–32 thread sweep.
- The resulting time-versus-threads curve shows an initial decrease in wall-clock time as threads increase, followed by saturation.
- The saturation point is consistent with the paper's finding that the implementation scaled out up to 32 threads (overhead cancels further gains beyond some thread count).
- The raw Zenodo data set was unzipped into the repository's data folder, so runs are reproducible.
- Exactly 100 randomly selected MS1 frames are used per run, matching the study protocol.

## Limitations

- The implementation scales out only up to the point where parallelization overhead cancels further speed gains; beyond that, added threads do not reduce wall-clock time.
- GPU acceleration (TensorFlow 2) is a separate capability and is not exercised by this CPU thread-scaling benchmark.
- The README notes the implementation has been shifted to the proteolizard framework, so the mzBucket repo's own build instructions are superseded by the proteolizard-data/algorithm repositories.

## Evidence

- [abstract] Our implementation scaled out up to 32 threads and supports acceleration by GPUs: "Our implementation scaled out up to 32 threads and supports acceleration by GPUs"
- [methods] output_scalability_study: [evidence span withheld — anchor status: not_found]
- [abstract] Raw data is available at https://zenodo.org/record/5036526: "Raw data is available at https://zenodo.org/record/5036526"
- [readme] The data set used is available for download ... Please unzip it into the data folder to ensure reproducible runs of the scripts.: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [readme] pyproteolizard-data for data-access: "more precisely the components [pyproteolizard-data](https://github.com/theGreatHerrLebert/proteolizard-data) for data-access and [pyproteolizard-algorithm](https://github.com/theGreatHerrLebert/proteo"
- [results] We built on top of the low-level C++ API of OpenTIMS [29] for data access.: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [results] our implementation supports acceleration by graphics processing units (GPUs) through the TensorFlow 2 software library: "our implementation supports acceleration by graphics processing units (GPUs) through the TensorFlow 2 software library [28]."
