---
name: parallel-batch-processing-of-large-metabolomics-studies
description: Use when an untargeted LC/HRMS study has hundreds or thousands of sample
  files (e.g. 499–1,649 mzML/mzXML runs) and sequential peak-picking would be too
  slow; you need to parallelize peak detection and m/z-slice alignment across threads.
when_to_use_negative:
- The study contains only a handful of samples and sequential peak picking with MZmine
  2, xcms, or MS-DIAL is fast enough — parallelization overhead is unnecessary.
- The goal is MS2/DDA or DIA fragmentation processing rather than MS1 peak detection
  and alignment; use a fragmentation-focused workflow instead.
- Input is already an aligned feature table that only needs downstream steps like
  adduct/fragment identification or molecular formula annotation (e.g. by IDSL.UFA).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: R package that implements parallel peak detection and alignment; thread count
    is set in its parameter spreadsheet and run via the IPA_workflow function
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Runtime environment in which IDSL.IPA is installed and the workflow is executed
- name: MS-DIAL
  role: Comparator tool used to validate the shared-peak coverage of the parallelized
    alignment output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:4b161e5b7a25ea354f226f6c59299fb9b82c0ad439b08092f83110181cb6e29e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Parallel batch processing of large metabolomics studies

## Summary

A method for accelerating untargeted LC/HRMS peak processing of population-scale studies (hundreds to thousands of samples) by distributing peak detection and alignment across multiple computing threads, as implemented in the IDSL.IPA R package.

## When to use

Use when an untargeted LC/HRMS study has hundreds or thousands of sample files (e.g. 499–1,649 mzML/mzXML runs) and sequential peak-picking would be too slow; you need to parallelize peak detection and m/z-slice alignment across threads.

## When NOT to use

- The study contains only a handful of samples and sequential peak picking with MZmine 2, xcms, or MS-DIAL is fast enough — parallelization overhead is unnecessary.
- The goal is MS2/DDA or DIA fragmentation processing rather than MS1 peak detection and alignment; use a fragmentation-focused workflow instead.
- Input is already an aligned feature table that only needs downstream steps like adduct/fragment identification or molecular formula annotation (e.g. by IDSL.UFA).

## Inputs

- 499 or more untargeted LC/HRMS data files in mzML, mzXML, or netCDF format (e.g. the MTBLS1684 cord blood study)
- Filled IDSL.IPA parameter spreadsheet with input/output locations and the number of processing threads (e.g. 20)
- RT correction inputs: endogenous retention-index markers for the dynamic retention-index marker approach

## Outputs

- Aligned peak intensity matrix (peak table) with on the order of ~214,000 rows for a 499-sample study, saved as aligned_peak_table.csv
- Per-sample peaklists containing m/z, RT, and chromatographic peak properties
- Run metrics: total aligned peaks, processing time, and shared-peak overlap vs. MS-DIAL (e.g. 82%, 41,172 peaks)

## How to apply

Set the number of processing threads in the IDSL.IPA parameter spreadsheet (e.g. 20 threads for the 499-sample MTBLS1684 cord blood study) and run the IPA_workflow function, which executes sections 1–6: 12C/13C isotopologue pair detection in each scan, EIC construction, chromatographic peak characterization, dynamic retention-index-marker RT correction, and parallel peak alignment across m/z slices. The rationale is that peak detection operates independently per file and alignment operates independently per m/z cluster, so each unit can be dispatched to a separate thread without cross-dependencies. A comparable run processed 499 samples into a ~214,000-peak aligned table within 1 hour using 20 parallel threads, and the pipeline has been evaluated on studies of 200–1,600 samples.

## Related tools

- **IDSL.IPA** (R package that implements parallel peak detection and alignment; thread count is set in its parameter spreadsheet and run via the IPA_workflow function) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Runtime environment in which IDSL.IPA is installed and the workflow is executed)
- **MS-DIAL** (Comparator tool used to validate the shared-peak coverage of the parallelized alignment output)

## Examples

```
library(IDSL.IPA); IPA_workflow("path/to/IPA_parameters.xlsx")
```

## Evaluation signals

- Aligned peak table exists, is a CSV, and contains the expected row scale (~214,000 aligned peaks for 499 samples in the MTBLS1684 study).
- Parallel run completes within roughly 1 hour for a 499-sample study using 20 threads; scaling beyond this without hitting single-threaded bottlenecks indicates the parallelization is active.
- Shared-peak overlap against an MS-DIAL run reaches ~82% (41,172 shared peaks), confirming the parallel path did not lose sensitivity.
- Peaks are distributed only across non-overlapping m/z clusters (each 12C m/z peak used in exactly one cluster), avoiding duplicate intensity contributions.
- Aligned matrices are further processed for adduct/fragment identification, statistics filtering, and formula annotation (e.g. by IDSL.UFA) without re-running peak detection.

## Limitations

- Aligned IDSL.IPA matrices still require downstream processing to identify adducts and fragments, apply statistical filtering, and annotate molecular formulas.
- A majority of rejected EICs in validation were low-abundance peaks that passed the S/N threshold but lacked isotopologue pairing; additional auto-filtering rules remain a need.
- Low-abundance, isotopologue-unpaired peaks that passed S/N alone are not always recovered, so sensitivity on trace analytes may be lower than expected.
- Thread count must be set explicitly in the parameter spreadsheet (e.g. 20); default settings may under-utilize available compute.
- Pipeline is designed for organic/carbon-containing compound detection (12C/13C ion pairing); non-carbon analytes fall outside its targeted signal space.

## Evidence

- [readme] The README claims parallelized IDSL.IPA outperforms competing peak pickers in speed: [evidence span withheld — anchor status: external_doc]
