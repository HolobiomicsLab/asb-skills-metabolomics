---
name: zone-of-interest-derivation
description: Use when you need to reproduce Paramounter's automatic parameter-measurement
  step from raw LC-MS intensity profiles by reconstructing rank-based intensity sorting
  and the zone-of-interest mechanism in R.
when_to_use_negative:
- Input is already a feature table or processed peak list rather than raw LC-MS intensity
  data — zone-of-interest derivation operates on raw intensity profiles prior to feature
  extraction.
- You only need to run Paramounter end-to-end for parameter tuning rather than reconstruct/validate
  the zone-of-interest mechanism itself.
- Data are not LC-MS-based metabolomics profiles (e.g., GC-MS, NMR, or non-spectrometric
  data), which the article's scope does not cover.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: Paramounter
  role: R program whose deposited implementation defines the reference zone-of-interest
    derivation; its module is run to produce artifacts for comparison
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: implementation language used to load raw intensity profiles, perform rank-based
    sorting, and derive the zone-of-interest artifacts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:9b8ee334b726c387385fb8acbaf276a58ad4c007dedf01319a539a06ea017c59
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - Paramounter
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# zone-of-interest-derivation

## Summary

Derives Paramounter's zone-of-interest structure from a rank-based descending sorted intensity profile of raw LC-MS data, which underpins automatic measurement of universal feature-extraction parameters (mass tolerance, peak height, peak width, instrumental shift).

## When to use

Use when you need to reproduce Paramounter's automatic parameter-measurement step from raw LC-MS intensity profiles by reconstructing rank-based intensity sorting and the zone-of-interest mechanism in R.

## When NOT to use

- Input is already a feature table or processed peak list rather than raw LC-MS intensity data — zone-of-interest derivation operates on raw intensity profiles prior to feature extraction.
- You only need to run Paramounter end-to-end for parameter tuning rather than reconstruct/validate the zone-of-interest mechanism itself.
- Data are not LC-MS-based metabolomics profiles (e.g., GC-MS, NMR, or non-spectrometric data), which the article's scope does not cover.

## Inputs

- raw LC-MS intensity profile for a single sample file (R vector/matrix of intensity values)
- deposited HuanLab/Paramounter R source (https://github.com/HuanLab/Paramounter)

## Outputs

- sorted-intensity structure artifact saved as CSV
- zone-of-interest structure artifact saved as TXT/CSV
- reference-matched output for the corresponding Paramounter module

## How to apply

Load the raw LC-MS intensity profile for a single sample file and sort the extracted intensity values in rank-based descending order to build the sorted-intensity structure used by Paramounter. From this rank-ordered profile, derive the zone-of-interest structure as described in the method narrative, then write the sorted-intensity artifact to CSV and the zone-of-interest artifact to TXT/CSV. Run the corresponding module of the deposited HuanLab/Paramounter R implementation on the same input and compare intermediate artifacts to a reference run, accepting the implementation only when outputs match (output_matches_reference). The derived zone of interest feeds Paramounter's measurement of universal parameters (mass tolerance, peak height, peak width, instrumental shift) that are subsequently translated to software-specific parameters for tools like MS-DIAL and XCMS CentWave.

## Related tools

- **Paramounter** (R program whose deposited implementation defines the reference zone-of-interest derivation; its module is run to produce artifacts for comparison) — https://github.com/HuanLab/Paramounter
- **R** (implementation language used to load raw intensity profiles, perform rank-based sorting, and derive the zone-of-interest artifacts)

## Examples

```
Rscript Paramounter.R --input raw_intensity.csv --sorted sorted_intensity.csv --zone zone_of_interest.csv
```

## Evaluation signals

- Sorted-intensity CSV artifact matches the rank-based descending ordering of the input raw LC-MS intensities.
- Zone-of-interest artifact (TXT/CSV) is byte- or value-identical to the corresponding artifact from the deposited HuanLab/Paramounter run on the same input.
- Running the deposited Paramounter module on the same input reproduces the reference example run (output_matches_reference).
- Downstream universal parameters (mass tolerance, peak height, peak width, instrumental shift) derived from the zone of interest remain consistent with the reference run.
- Feature-extraction results using the derived parameters show the expected uplift (e.g., threefold increase vs. MS-DIAL defaults, 3.7-/1.6-fold more true positive features than AutoTuner/IPO).

## Limitations

- Paramounter's optimized peak height yields a higher rate of false positive features and a likelihood of software crash; users may need a higher peak height threshold (e.g., 2X the optimized value).
- mzdiff dereplication with suggested defaults (0.001 or 0.01) can mistakenly remove true positive features with smaller mass differences; set mzdiff to a negative value to disable dereplication.
- The abstract only briefly references the zone-of-interest concept — detailed derivation logic must be reconstructed from the deposited implementation.

## Evidence

- [abstract] rank-based intensity sorting and zone of interest: [evidence span withheld — anchor status: not_found]
- [other] workflow artifacts sorted-intensity CSV and zone-of-interest TXT/CSV compared to reference run: [evidence span withheld — anchor status: not_found]
