---
name: multithreaded-wall-clock-timing-analysis
description: Use when benchmarking a parallel raw-data-processing implementation (e.g.
  LSH signal classification on MS1 TimsTOF frames) to quantify how wall-clock time
  changes with thread count and find the point where speedups saturate.
when_to_use_negative:
- When the goal is to measure GPU-accelerated performance via TensorFlow 2 — this
  skill benchmarks CPU thread scaling only, and the README notes the implementation
  has been shifted to the proteolizard framework rather than a standalone build.
- When evaluating classification quality (ROC curves, true/false-positive rates for
  (m,n) settings like (32,16), (64,32), (32,32)) — use the roc/createSynthetics workflows
  instead; this skill measures only timing, not correctness.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: pyproteolizard-data
  role: Provides the OpenTIMS/PyTimsDataHandle data-access layer used to load the
    TimsTOF raw data and select the 100 MS1 frames for the timing sweep.
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Supplies the LSH implementation that is executed at each thread count during
    the scalability sweep.
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: mzBucket
  role: Repository holding the scalability-study scripts, output_scalability_study
    folder, and plots directory in which results and figures are written.
  repo: https://github.com/hildebrandtlab/mzBucket
- name: Python
  role: Language in which the data access, LSH runs, and timing aggregation are driven
    (C++ backends are exposed via pybind11).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:256af7a4b5f75cd184acef38d3a4fd50e3af0f14ad4392a6e60f8ce6adb7e46c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - C++
  - OpenTIMS
  - Python
  - TensorFlow 2
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# multithreaded-wall-clock-timing-analysis

## Summary

Measures how the wall-clock execution time of the mzBucket/proteolizard LSH signal-classification implementation on real TimsTOF frames scales as the thread count is swept from 1 to 32, using the OpenTIMS/pyproteolizard data-access layer on CPU only.

## When to use

Use when benchmarking a parallel raw-data-processing implementation (e.g. LSH signal classification on MS1 TimsTOF frames) to quantify how wall-clock time changes with thread count and find the point where speedups saturate.

## When NOT to use

- When the goal is to measure GPU-accelerated performance via TensorFlow 2 — this skill benchmarks CPU thread scaling only, and the README notes the implementation has been shifted to the proteolizard framework rather than a standalone build.
- When evaluating classification quality (ROC curves, true/false-positive rates for (m,n) settings like (32,16), (64,32), (32,32)) — use the roc/createSynthetics workflows instead; this skill measures only timing, not correctness.

## Inputs

- Raw TimsTOF DDA-PASEF acquisition from Zenodo record 5036526 (Bruker .d folder)
- 100 randomly selected MS1 frames of the acquisition
- pyproteolizard-algorithm LSH implementation and thread count parameter (1–32)

## Outputs

- scalability_results.csv in data/output_scalability_study (wall-clock time vs. thread count table)
- timing/speedup plot saved in the plots directory

## How to apply

Download the raw DDA-PASEF data set from Zenodo record 5036526 and unzip it into the data folder of the mzBucket repository so scripts run reproducibly. Open the TimsTOF data via the pyproteolizard-data/OpenTIMS PyTimsDataHandle layer, randomly select 100 MS1 (precursor) frames, and run the LSH algorithm from pyproteolizard-algorithm on those frames repeatedly. Sweep the thread count from 1 up to 32 threads on CPU (no TensorFlow 2 GPU acceleration is needed for this study), recording the wall-clock execution time at each thread count. Aggregate the timings into a wall-clock-time-versus-threads table written to output_scalability_study as scalability_results.csv, then plot the timing curve and save it under the plots directory. Validate that the curve first decreases and then saturates, i.e. parallelization overhead cancels further gains, matching the paper's finding that the implementation scaled out up to 32 threads.

## Related tools

- **pyproteolizard-data** (Provides the OpenTIMS/PyTimsDataHandle data-access layer used to load the TimsTOF raw data and select the 100 MS1 frames for the timing sweep.) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Supplies the LSH implementation that is executed at each thread count during the scalability sweep.) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **mzBucket** (Repository holding the scalability-study scripts, output_scalability_study folder, and plots directory in which results and figures are written.) — https://github.com/hildebrandtlab/mzBucket
- **Python** (Language in which the data access, LSH runs, and timing aggregation are driven (C++ backends are exposed via pybind11).)

## Examples

```
python src/python/scalability/run_scalability.py --threads 1 2 4 8 16 32 --frames 100 --output data/output_scalability_study/scalability_results.csv
```

## Evaluation signals

- The timing curve decreases monotonically at low thread counts and then saturates at high thread counts, consistent with the reported 'scaled out up to 32 threads' behavior.
- scalability_results.csv contains a timing row for every swept thread count from 1 to 32.
- The same 100 randomly selected MS1 frames are used for every thread count, so per-thread timings are directly comparable.
- The plot is saved under the plots directory and the CSV under data/output_scalability_study, matching the mzBucket repo layout.
- Wall-clock time at 32 threads is lower than at 1 thread but not dramatically lower, indicating parallelization overhead has begun to cancel further gains.

## Limitations

- Only CPU thread scaling up to 32 threads is reported; the study does not sweep beyond 32 threads.
- GPU acceleration via TensorFlow 2 is supported by the implementation but is not part of this timing study.
- Wall-clock timings include parallelization overhead, so the saturation point conflates hardware scheduling with algorithmic scaling.

## Evidence

- [readme] Zenodo record download and placement in data folder: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
- [readme] Proteolizard components used for data access and the LSH: "more precisely the components [pyproteolizard-data](https://github.com/theGreatHerrLebert/proteolizard-data) for data-access and [pyproteolizard-algorithm](https://github.com/theGreatHerrLebert/proteo"
- [abstract] Thread-scaling finding: "Our implementation scaled out up to 32 threads and supports acceleration by GPUs"
- [results] GPU acceleration is available through TensorFlow 2: "our implementation supports acceleration by graphics processing units (GPUs) through the TensorFlow 2 software library"
- [results] OpenTIMS low-level data access: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
