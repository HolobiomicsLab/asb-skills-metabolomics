---
name: lcms-intensity-rank-sorting
description: Use when processing raw LC-MS-based metabolomics data in R to derive
  universal feature-extraction parameters via rank-ordered intensity profiles and
  zone-of-interest structures, prior to tools like MS-DIAL or XCMS CentWave.
when_to_use_negative:
- Input is already a processed feature table or peak list — rank sorting and zone-of-interest
  derivation apply to raw LC-MS intensity profiles only.
- Dereplication of features is not needed — mzdiff should be set to a negative value
  instead of relying on this skill's parameter tuning.
- Goal is downstream statistical analysis of metabolites — this skill only measures
  universal parameters prior to feature extraction.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Paramounter
  role: R program implementing rank-based intensity sorting and zone-of-interest derivation
    to measure universal parameters from raw LC-MS data
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Runtime environment for loading intensity profiles and executing Paramounter
    modules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:50e35895807c85c0e2454914b548b5c6332cc87c5e6bdbae5477010562392b6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - Paramounter
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lcms-intensity-rank-sorting

## Summary

Reconstructs Paramounter's rank-based intensity sorting and zone-of-interest mechanism, which underlies its automatic measurement of universal LC-MS parameters (mass tolerance, peak height, peak width, instrumental shift) from raw metabolomics data in R.

## When to use

Use when processing raw LC-MS-based metabolomics data in R to derive universal feature-extraction parameters via rank-ordered intensity profiles and zone-of-interest structures, prior to tools like MS-DIAL or XCMS CentWave.

## When NOT to use

- Input is already a processed feature table or peak list — rank sorting and zone-of-interest derivation apply to raw LC-MS intensity profiles only.
- Dereplication of features is not needed — mzdiff should be set to a negative value instead of relying on this skill's parameter tuning.
- Goal is downstream statistical analysis of metabolites — this skill only measures universal parameters prior to feature extraction.

## Inputs

- Raw LC-MS-based metabolomics intensity profile for a single sample file
- R environment with the HuanLab/Paramounter implementation

## Outputs

- Sorted-intensity artifact (CSV)
- Zone-of-interest artifact (TXT/CSV)
- Universal parameter measurements (mass tolerance, peak height, peak width, instrumental shift)

## How to apply

Load a raw LC-MS intensity profile for a single sample file into R and sort the extracted intensity values in rank-based descending order to build the sorted-intensity structure used by Paramounter. From this rank-ordered profile, derive the zone-of-interest structure per the method narrative. Write the sorted-intensity artifact to CSV and the zone-of-interest artifact to TXT/CSV, then run the corresponding module of the deposited HuanLab/Paramounter implementation on the same input and compare intermediate artifacts against the deposited example run. Accept the implementation only when produced artifacts match the reference run (output_matches_reference); the resulting parameters then feed software-specific tuning, where mzdiff defaults of 0.001 or 0.01 control dereplication of XCMS CentWave features and peak height thresholds (e.g., 2X the optimized value) reduce false positives.

## Related tools

- **Paramounter** (R program implementing rank-based intensity sorting and zone-of-interest derivation to measure universal parameters from raw LC-MS data) — https://github.com/HuanLab/Paramounter
- **R** (Runtime environment for loading intensity profiles and executing Paramounter modules)

## Examples

```
Rscript Paramounter_rank_sorting.R --input raw_intensity.csv --sorted-output sorted_intensities.csv --zone-output zone_of_interest.csv
```

## Evaluation signals

- Sorted-intensity and zone-of-interest artifacts produced locally must match the deposited HuanLab/Paramounter example run for the same input (output_matches_reference).
- Sorted-intensity output is written as CSV and zone-of-interest output as TXT/CSV per the documented workflow.
- The derived zone-of-interest structure is consistent with a rank-based descending ordering of the raw intensity values.
- Resulting universal parameters (mass tolerance, peak height, peak width, instrumental shift) translate to valid software-specific parameters for MS-DIAL or XCMS CentWave feature extraction.
- Applying the measured parameters yields a threefold increase in extracted metabolites compared to default MS-DIAL parameters.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but incurs a higher rate of false positives and software-crash likelihood; users may need a higher threshold (e.g., 2X the optimized value).
- mzdiff dereplication defaults (0.001 or 0.01) can remove true positive metabolic features with mass differences smaller than that value; set mzdiff to any negative value to disable dereplication.
- Validation covered 11 datasets across sample types, data-acquisition modes, and MS vendors, but per-dataset behavior may still vary.

## Evidence

- [methods] rank-based intensity sorting and zone of interest: [evidence span withheld — anchor status: not_found]
- [methods] Load the raw LC-MS intensity profile: [evidence span withheld — anchor status: not_found]
- [methods] sort extracted intensity values in rank-based descending order: [evidence span withheld — anchor status: not_found]
- [methods] derive the zone-of-interest structure: [evidence span withheld — anchor status: not_found]
- [methods] write sorted-intensity artifact to CSV: [evidence span withheld — anchor status: not_found]
