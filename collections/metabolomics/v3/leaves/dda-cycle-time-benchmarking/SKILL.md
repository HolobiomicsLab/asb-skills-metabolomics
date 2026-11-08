---
name: dda-cycle-time-benchmarking
description: Use when evaluating how DDA acquisition speed (cycle time) affects feature-extraction
  depth, or when validating that MS2-recognition rescue in JPA compensates for PP
  feature loss on DDA mzXML data.
when_to_use_negative:
- Input data are full-scan or DIA acquisitions — JPA explicitly warns not to use find.level3features
  on such datasets
- The question is feature annotation or alignment quality rather than feature-count
  robustness across acquisition cycle times
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: R package providing XCMS.featureTable() for MS1 PP extraction and find.level3features()
    for MR feature rescue, used to compare feature counts per cycle time
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded XCMS peak-picking algorithm inside JPA extracts PP features from
    mzXML with parameters such as ppm = 10 and peakwidth = c(5, 20)
- name: R
  role: Runtime environment (version 4.0.0 or above) in which JPA is installed and
    the benchmark is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:7f27937d0117f8df55e9420609ad7a80649e2839679bc5db093e4967a9c9c1de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dda-cycle-time-benchmarking

## Summary

Benchmarks JPA-PP (XCMS peak-picking) versus JPA-MR (MS2-recognition) feature counts across DDA cycle times (1, 2, 3 s) to show that MR rescues features lost as MS2 duty cycle slows.

## When to use

Use when evaluating how DDA acquisition speed (cycle time) affects feature-extraction depth, or when validating that MS2-recognition rescue in JPA compensates for PP feature loss on DDA mzXML data.

## When NOT to use

- Input data are full-scan or DIA acquisitions — JPA explicitly warns not to use find.level3features on such datasets
- The question is feature annotation or alignment quality rather than feature-count robustness across acquisition cycle times

## Inputs

- DDA mzXML files for 1 s, 2 s, and 3 s cycle-time acquisitions (MetaboLights MTBLS2631), each cycle time in its own folder
- R environment with JPA (R >= 4.0.0) and embedded xcms functions installed

## Outputs

- Per-cycle-time feature tables with 'level' column labeling features as 'PP' or 'MR'
- Summary table of PP and MR feature counts per DDA cycle time (1 s, 2 s, 3 s)

## How to apply

Download the DDA mzXML files for the 1 s, 2 s, and 3 s cycle-time acquisitions from MetaboLights accession MTBLS2631 and place each dataset's files in its own folder. For each folder, run JPA's XCMS.featureTable() to extract MS1 PP features with the embedded XCMS parameters (e.g., ppm = 10, peakwidth = c(5, 20), snthresh = 6, prefilter = c(3, 100)), then run find.level3features(data = MSdata, level3.threshold = 2) to add MR features — valid only because the data are DDA, not full-scan or DIA. Count features by the 'level' column values 'PP' and 'MR' per cycle time and assemble a summary table. Validate the trend from Figure S3: JPA-PP counts decrease monotonically and JPA-MR counts increase monotonically as cycle time increases from 1 s to 3 s, and flag any cycle time violating that pattern.

## Related tools

- **JPA** (R package providing XCMS.featureTable() for MS1 PP extraction and find.level3features() for MR feature rescue, used to compare feature counts per cycle time) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded XCMS peak-picking algorithm inside JPA extracts PP features from mzXML with parameters such as ppm = 10 and peakwidth = c(5, 20))
- **R** (Runtime environment (version 4.0.0 or above) in which JPA is installed and the benchmark is executed)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=10, peakwidth=c(5,20), snthresh=6, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data = MSdata, level3.threshold = 2)
```

## Evaluation signals

- PP feature counts decrease monotonically as cycle time increases from 1 s to 3 s, matching Figure S3
- MR feature counts increase monotonically as cycle time increases from 1 s to 3 s, matching Figure S3
- Feature table's 'level' column contains only 'PP' and 'MR' values after find.level3features()
- Each dataset folder contains only that cycle time's mzXML files, as JPA requires no irrelevant mzXML files in the input folder

## Limitations

- find.level3features() must not be applied to full-scan or DIA data — the MR rescue is only valid for DDA
- PP counts can vary with XCMS parameter settings (ppm, peakwidth, snthresh, prefilter), so parameters must be held constant across cycle times for a fair comparison
- Findings were validated on MTBLS2631 DDA data; trends may differ for other instruments or acquisition modes

## Evidence

- [readme] find.level3features valid only for DDA: "After PP features have been extracted, extracting features using MS2 recognition (labeled as "MR") can be performed. This step is optional. Please do not use this function when processing full-scan"
- [readme] XCMS.featureTable invocation and parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] level column labeling: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [supplementary] trend reproduced from task card (task_007 finding): [evidence span withheld — anchor status: not_found]
- [methods] embedded XCMS functions: [evidence span withheld — anchor status: not_found]
