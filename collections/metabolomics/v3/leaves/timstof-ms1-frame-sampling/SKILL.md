---
name: timstof-ms1-frame-sampling
description: Use when you need to run the LSH signal-classification algorithm (mzBucket/proteolizard-algorithm)
  on a subset of a TimsTOF acquisition, or benchmark its runtime as the thread count
  is swept, without loading the full DDA-PASEF data set into memory.
when_to_use_negative:
- Do not sample MS1 frames when the analysis target is the vendor-selected DDA-PASEF
  fragmentation subset — those precursor peaks (N=6902 vendor-selected peptides in
  the study subset) live in fragmented frames and require the fragment_frames ids
  instead.
- Do not use frame sampling when a gold-standard peak-level annotated ground truth
  is required; the article notes no such annotation exists for the real data, so classification-rate
  evaluation relies on synthetic data instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: proteolizard-data (PyTimsDataHandle)
  role: Object-oriented data-access layer built on OpenTIMS' low-level C++ API; PyTimsDataHandle
    reads frame metadata, exposes dh.precursor_frames and dh.fragment_frames arrays,
    and returns individual frames as sparse tables
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API through which raw TimsTOF data is accessed for frame-level
    loading
- name: pybind11
  role: Interfaces the C++ proteolizard-data classes with Python so frames can be
    fetched as pandas/numpy tables in Python workflows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:e0ec8a6ca56ca3fd77ececa3a888020066dab603ee56de463ec268672551d453
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
  - proteolizard-data (PyTimsDataHandle)
  - pybind11
  - pyproteolizard-algorithm
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# timstof-ms1-frame-sampling

## Summary

Samples a small, reproducible subset of MS1 precursor frames from a Bruker TimsTOF .d acquisition (via the proteolizard-data/OpenTIMS access layer) so that expensive LSH signal-classification runs, scalability sweeps, and algorithm-prototyping experiments operate on manageable slices of the sparse retention-time/ion-mobility/m/z tensor.

## When to use

Use when you need to run the LSH signal-classification algorithm (mzBucket/proteolizard-algorithm) on a subset of a TimsTOF acquisition, or benchmark its runtime as the thread count is swept, without loading the full DDA-PASEF data set into memory.

## When NOT to use

- Do not sample MS1 frames when the analysis target is the vendor-selected DDA-PASEF fragmentation subset — those precursor peaks (N=6902 vendor-selected peptides in the study subset) live in fragmented frames and require the fragment_frames ids instead.
- Do not use frame sampling when a gold-standard peak-level annotated ground truth is required; the article notes no such annotation exists for the real data, so classification-rate evaluation relies on synthetic data instead.

## Inputs

- Bruker TimsTOF .d raw folder (TDF format, Zenodo record 5036526 data set) placed in the mzBucket data folder
- SQLite frame metadata embedded in the .d folder (read automatically by PyTimsDataHandle)

## Outputs

- A randomly selected subset of 100 MS1 precursor frame ids from the acquisition
- Per-frame sparse data tables (frame, scan, mz, intensity) retrieved via data_handle.get_frame(id)

## How to apply

First download the raw data set from Zenodo record 5036526 and unzip it into the data folder of the mzBucket repository, since the scripts expect reproducible paths. Construct a PyTimsDataHandle on the .d folder; on construction it reads the acquisition-generated SQLite metadata table (Id, Time, MsMsType, NumScans, NumPeaks, AccumulationTime, etc.), which also exposes two numpy arrays, dh.precursor_frames and dh.fragment_frames, holding all MS1 and MS2 frame ids respectively. Sample 100 frames randomly from the MS1 (precursor) frame ids — the article's scalability study uses exactly 100 randomly selected MS1 frames from the acquisition — and load each with data_handle.get_frame(id) as sparse (frame, scan, m/z, intensity) tables. This keeps memory bounded while preserving the sparse 3D structure needed for windowed locality-sensitive hashing on the mass axis. The sampled subset then feeds directly into LSH runs (e.g. thread-count sweeps from 1 to 32) whose timings are written to scalability_results.csv under output_scalability_study.

## Related tools

- **proteolizard-data (PyTimsDataHandle)** (Object-oriented data-access layer built on OpenTIMS' low-level C++ API; PyTimsDataHandle reads frame metadata, exposes dh.precursor_frames and dh.fragment_frames arrays, and returns individual frames as sparse tables) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API through which raw TimsTOF data is accessed for frame-level loading)
- **pybind11** (Interfaces the C++ proteolizard-data classes with Python so frames can be fetched as pandas/numpy tables in Python workflows)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('data/experiment.d'); ids = dh.precursor_frames; sample = np.random.choice(ids, 100, replace=False); frames = [dh.get_frame(i) for i in sample]
```

## Evaluation signals

- Only frame ids present in dh.precursor_frames are sampled; no fragmented (MsMsType 8) frame ids appear in the selected subset.
- The sampled subset size matches the article's protocol — 100 MS1 frames — and each get_frame(id) call returns a non-empty (frame, scan, mz, intensity) table consistent with the NumScans/NumPeaks values in the metadata table.
- All runs on the sampled frames use the same random selection so timing sweeps across 1–32 threads are comparable; timings monotonically decrease then saturate as thread count increases.
- The .d folder was unzipped into the repository's data folder (per README) so PyTimsDataHandle construction succeeds reproducibly.

## Limitations

- MS1 precursor frames alone cannot fully reflect the behavior of real DDA-PASEF data: vendor-selected DDA peaks form a biased reference, corresponding only to peptides selected for fragmentation and likely stemming from high-intensity signals.
- The additional ion-mobility dimension and increased data sparsity of TimsTOF raw data make the sampled frames large 3D sparse tensors; processing full frames can be slow, which is precisely why a small random subset is used.
- No gold-standard peak-level ground-truth annotation was available for the real data, so sampled-frame outputs cannot be validated against a known truth without synthetic reference patterns.

## Evidence

- [methods] Random MS1 frame sampling for LSH runs: [evidence span withheld — anchor status: not_found]
- [results] Data-access layer built on OpenTIMS C++ API: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [readme] Raw data location and unzip instruction: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [readme] Data handle exposes precursor and fragment frame id arrays: [evidence span withheld — anchor status: external_doc]
- [readme] Frame as smallest unit of raw data: [evidence span withheld — anchor status: external_doc]
- [readme] Sparse 3D tensor nature of TimsTOF raw data: [evidence span withheld — anchor status: external_doc]
- [readme] PyTimsDataHandle reads acquisition-generated SQLite metadata: [evidence span withheld — anchor status: external_doc]
- [discussion] Biased nature of vendor-selected reference peaks: "the annotated masses correspond only to peptides selected for fragmentation and, second, those peptides likely stem from high-intensity signals."
