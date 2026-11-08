---
name: binning-output-quality-control
description: Use when a bucket table produced by NMRProcFlow binning (Intelligent,
  variable-size or fixed-width) contains buckets with weak signal, and buckets must
  be retained or discarded by signal-to-noise ratio (SNR) before downstream statistics.
when_to_use_negative:
- The task is upstream spectra processing — ppm-scale calibration, baseline correction,
  or realignment of ppm areas — rather than quality control of the binning output
- 'Bucketing has not yet been performed: this skill operates on the bucket table produced
  by binning, not on raw FID spectra'
- You need to quantify metabolites in crowded regions with heavy peak overlap; the
  paper plans local signal deconvolution for that in a future version, not SNR filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: NMRProcFlow
  role: Interactive GUI tool whose binning module (Adaptive Intelligent Binning, variable-size
    or fixed-width) produces the bucket table and which supports SNR-based bucket
    filtering during the computing process or at the output step
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Implementation language of NMRProcFlow's processing core (NMRspec); usable
    to script SNR computation and bucket filtering on the bucket table outside the
    GUI
- name: Docker
  role: Container runtime used to deploy NMRProcFlow locally so the bucketing and
    SNR-filtering workflow can be run on your own data
  repo: docker.io/nmrprocflow/npflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:b951df862ec2d77bf025de264f4c36c67c0e39f4b6afcf62964b4040c3c6bc48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# binning-output-quality-control

## Summary

Quality-controls the bucket table produced by NMR binning in NMRProcFlow by retaining or discarding buckets according to their signal-to-noise ratio (SNR), applicable either during the computing process or at the output step. This removes bins whose signal is indistinguishable from spectral noise before downstream metabolomics statistics.

## When to use

Use when a bucket table produced by NMRProcFlow binning (Intelligent, variable-size or fixed-width) contains buckets with weak signal, and buckets must be retained or discarded by signal-to-noise ratio (SNR) before downstream statistics.

## When NOT to use

- The task is upstream spectra processing — ppm-scale calibration, baseline correction, or realignment of ppm areas — rather than quality control of the binning output
- Bucketing has not yet been performed: this skill operates on the bucket table produced by binning, not on raw FID spectra
- You need to quantify metabolites in crowded regions with heavy peak overlap; the paper plans local signal deconvolution for that in a future version, not SNR filtering

## Inputs

- Bucket table from NMRProcFlow binning: ppm intervals with per-spectrum intensities (Adaptive Intelligent Binning, variable-size or fixed-width bins)
- Processed 1D 1H or 13C NMR spectra (FID from Bruker or Agilent/Varian) supplying the spectral noise level used for SNR estimation

## Outputs

- Filtered bucket table (filtered_bucket_table.csv) listing retained buckets with their ppm ranges and per-spectrum intensities

## How to apply

Load the bucket table produced by NMRProcFlow's binning stage (Adaptive Intelligent Binning per De Meyer et al. 2008, variable-size or fixed-width bins), where each row is a ppm interval carrying per-spectrum intensities. Compute the signal-to-noise ratio of every bucket from its intensity values relative to the spectral noise level of the processed 1D 1H or 13C spectra. Apply the SNR filter in NMRProcFlow either during the binning computing process or at the output step — the output-step mode is the right choice when the bucket table has already been computed. Discard every bucket whose SNR falls below the chosen threshold and retain the rest; the paper names no numeric cutoff, so set the threshold from the noise level of the spectra. Write the retained buckets with their ppm ranges and intensities to filtered_bucket_table.csv. The rationale is that equal-sized bins can lose information and create peak-shift artifacts, while SNR filtering removes bins whose signal is indistinguishable from noise before statistical analysis.

## Related tools

- **NMRProcFlow** (Interactive GUI tool whose binning module (Adaptive Intelligent Binning, variable-size or fixed-width) produces the bucket table and which supports SNR-based bucket filtering during the computing process or at the output step) — https://github.com/INRA/NMRProcFlow
- **R** (Implementation language of NMRProcFlow's processing core (NMRspec); usable to script SNR computation and bucket filtering on the bucket table outside the GUI)
- **Docker** (Container runtime used to deploy NMRProcFlow locally so the bucketing and SNR-filtering workflow can be run on your own data) — docker.io/nmrprocflow/npflow

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- filtered_bucket_table.csv exists and parses as a valid CSV
- Row count of the filtered bucket table equals the number of buckets whose computed SNR exceeds the chosen threshold
- Every retained bucket's ppm interval is present unchanged in the input bucket table (no new, merged, or split bins introduced)
- Every bucket removed by the filter has SNR below the threshold when recomputed from bucket intensities and the spectral noise level
- Applying the filter during the computing process versus at the output step with the same threshold yields the same retained bucket set

## Limitations

- The paper specifies no default SNR cutoff; the threshold must be chosen by the user and applied either during the computing process or at the output step
- NMRProcFlow is an interactive GUI tool; the article notes that batch-mode black-box processing usable by non-experts has not reached full maturity, so the filter is applied through the interface or via custom R scripting rather than a ready-made batch command
- Upstream spectra alignment requires user expertise (as noted in the article citing Larive et al. 2015); misaligned spectra can distort bucket intensities and therefore SNR estimates

## Evidence

- [results] SNR-based bucket filtering is a supported NMRProcFlow capability: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [methods] Binning modes that produce the bucket table being filtered: "and the binning (Intelligent, variable size or fixed width)"
- [results] Adaptive Intelligent Binning identifies the bin edges behind the bucket table: "Therefore, we implemented the Adaptive Intelligent Binning (De Meyer et al. 2008) algorithm which largely circumvents these problems by recursively identifying bin edges in existing bins."
- [readme] NMRProcFlow is the GUI tool hosting the spectra processing workflow: "The NMRProcFlow open source software provides an efficient GUI tool for spectra processing from 1D NMR metabolomics data"
