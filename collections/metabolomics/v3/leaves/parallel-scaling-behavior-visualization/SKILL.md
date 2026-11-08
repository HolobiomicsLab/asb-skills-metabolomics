---
name: parallel-scaling-behavior-visualization
description: Use when a multicore-capable implementation must be benchmarked for thread
  scalability — e.g. you need to know whether wall-clock time keeps decreasing or
  saturates as threads increase from 1 to 32 on real raw-data subsets.
when_to_use_negative:
- When the goal is GPU-acceleration benchmarking — the article's scaling study is
  a CPU thread sweep and explicitly does not require the TensorFlow 2 GPU path
- When you need classification accuracy metrics (ROC, true/false-positive rates) rather
  than performance timing — use the roc pipeline instead of the scalability study
- When data access itself is the bottleneck to benchmark — this skill isolates compute
  scaling on a fixed 100-frame subset, not I/O throughput
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: mzBucket
  role: Reference implementation whose threading behavior is measured; repository
    hosts the scalability scripts under src/python/scalability and the output folder
    data/output_scalability_study
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: OpenTIMS/pybind11-based data-access layer used to load the TimsTOF acquisition
    and select 100 MS1 frames
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Supplies the LSH signal-classification implementation whose wall-clock time
    is swept over thread counts
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: OpenTIMS
  role: Low-level C++ API underlying the data-access layer for reading bruker TDF
    raw data
  repo: https://github.com/michalsta/opentims
- name: pybind11
  role: C++/Python interface used by the proteolizard components; multicore C++ execution
    is what the thread sweep exercises
  repo: https://github.com/pybind/pybind11
- name: TensorFlow 2
  role: Optional GPU acceleration backend for the implementation; not needed for the
    CPU thread-count scaling sweep
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:490bb06356714e1ae0de8af98be8f9dc036e7032de56cec37b001cbccc579b96
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Parallel-scaling behavior visualization

## Summary

Sweeps the thread count of a CPU-parallel implementation (here the LSH signal-classification pipeline in pyproteolizard-algorithm/mzBucket) over TimsTOF raw-data frames, records wall-clock execution time per thread count, and plots/exports the scaling curve to reveal where parallelization gains saturate.

## When to use

Use when a multicore-capable implementation must be benchmarked for thread scalability — e.g. you need to know whether wall-clock time keeps decreasing or saturates as threads increase from 1 to 32 on real raw-data subsets.

## When NOT to use

- When the goal is GPU-acceleration benchmarking — the article's scaling study is a CPU thread sweep and explicitly does not require the TensorFlow 2 GPU path
- When you need classification accuracy metrics (ROC, true/false-positive rates) rather than performance timing — use the roc pipeline instead of the scalability study
- When data access itself is the bottleneck to benchmark — this skill isolates compute scaling on a fixed 100-frame subset, not I/O throughput

## Inputs

- Raw bruker TimsTOF .d folder (Zenodo record 5036526), unzipped into the repository data folder
- Randomly selected 100 MS1 frames from the acquisition (via pyproteolizard-data PyTimsDataHandle)
- Thread-count sweep configuration, e.g. 1 to 32 threads

## Outputs

- scalability_results.csv in data/output_scalability_study (wall-clock time vs. number of threads)
- Scaling plot (wall-clock time versus thread count) written under the plots directory

## How to apply

Download the raw TimsTOF data set from Zenodo record 5036526 and unzip it into the repository's data folder so the scripts run reproducibly. Load the acquisition via the pyproteolizard-data/OpenTIMS data-access layer (PyTimsDataHandle on the bruker .d folder) and randomly select 100 MS1 frames, which keeps the workload small enough that the CPU sweep dominates over data loading. Run the LSH algorithm from pyproteolizard-algorithm on these 100 frames repeatedly while sweeping the thread count from 1 up to 32, recording the wall-clock execution time for each thread count; the GPU-accelerated TensorFlow 2 path is not needed for this CPU scaling study. Aggregate the timings into a wall-clock-time-versus-threads table written to data/output_scalability_study as scalability_results.csv, then plot the curve into the plots directory. Validation: the reported behavior is an initial decrease of wall-clock time with added threads followed by saturation — parallelization overhead cancels further speed gains beyond some thread count.

## Related tools

- **mzBucket** (Reference implementation whose threading behavior is measured; repository hosts the scalability scripts under src/python/scalability and the output folder data/output_scalability_study) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (OpenTIMS/pybind11-based data-access layer used to load the TimsTOF acquisition and select 100 MS1 frames) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Supplies the LSH signal-classification implementation whose wall-clock time is swept over thread counts) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **OpenTIMS** (Low-level C++ API underlying the data-access layer for reading bruker TDF raw data) — https://github.com/michalsta/opentims
- **pybind11** (C++/Python interface used by the proteolizard components; multicore C++ execution is what the thread sweep exercises) — https://github.com/pybind/pybind11
- **TensorFlow 2** (Optional GPU acceleration backend for the implementation; not needed for the CPU thread-count scaling sweep)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle
dh = PyTimsDataHandle('data/experiment.d')
frames = dh.get_frame(frame_id)  # select 100 MS1 frames, then sweep threads 1–32 in src/python/scalability
```

## Evaluation signals

- scalability_results.csv exists in data/output_scalability_study and contains one wall-clock time entry per thread count in the swept range
- The timing curve shows the expected qualitative shape: initial decrease in wall-clock time as threads are added, followed by saturation (flat plateau) at higher thread counts
- The same fixed workload (100 randomly chosen MS1 frames from the same .d folder) is used for every thread count, so differences are attributable to parallelism, not data volume
- The plot is present under the plots directory and its axes correspond to wall-clock time vs. number of threads
- Rerunning after re-downloading and unzipping the Zenodo data set reproduces the same scaling behavior (per README's reproducibility instruction)

## Limitations

- Scaling only held up to the saturation point: 'the implementation scales out well only up to the point where parallelization overhead cancels further speed gains' — extra threads beyond that give no additional speedup
- Findings depend on the chosen subset (100 MS1 frames) and data set; behavior on other acquisitions or frame counts may differ
- The article's discussion notes an alternative Big-Data route (Apache Spark) was proposed for many large data sets, implying single-machine thread scaling may not transfer to distributed settings

## Evidence

- [abstract] Thread sweep up to 32 threads: "Our implementation scaled out up to 32 threads and supports acceleration by GPUs"
- [methods] Wall-clock time vs. threads output location: "output_scalability_study"
- [readme] scalability scripts in repository layout: "├── createSynthetics
        ├── meta
        ├── roc
        └── scalability"
- [readme] Data placement for reproducible runs: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [readme] Threading as a design goal of the algorithm library: [evidence span withheld — anchor status: external_doc]
- [results] pybind11 interface for C++/Python: "we used pybind11 [30] to interface C++ with the Python programming language"
- [results] GPU backend via TensorFlow 2: "our implementation supports acceleration by graphics processing units (GPUs) through the TensorFlow 2 software library [28]."
- [results] Low-level data access via OpenTIMS: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [discussion] Proposed distributed alternative for many large data sets: "We propose an implementation using a Big Data framework, such as Apache Spark [31], to facilitate testing on many large data sets"
- [readme] Data handle construction on .d folder: [evidence span withheld — anchor status: external_doc]
