---
name: extracted-ion-chromatogram-matching
description: Use when a DDA-detected metabolite feature (from pooled QC samples analyzed
  in DDA mode) lacks an intensity entry in the feature table of a DIA-analyzed individual
  biological sample, and its abundance must be recovered from the DIA raw data.
when_to_use_negative:
- When the feature is already detected in the DIA feature table — the rescue step
  only targets DDA-unique features missing from DIA runs
- When the input is not paired DDA/DIA mzXML data from the same pooled-QC and biological-sample
  set, since EIC matching requires both acquisition modes of the same cohort
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: DaDIA.R
  role: 'R program that performs the feature-rescue mechanism: flags DDA-unique features
    missing in DIA runs, re-determines their intensities from DIA raw data via EIC
    matching, and exports the rescued-feature intensity table'
  repo: https://github.com/HuanLab/DaDIA.git
- name: XCMS
  role: Used by DaDIA.R for the peak-picking step (centWave function) that extracts
    metabolic features from both DDA and DIA files prior to EIC matching
  repo: https://github.com/HuanLab/DaDIA.git
- name: R
  role: Runtime environment (Version 4.0 or above) required to execute DaDIA.R
- name: metaMS
  role: Required dependency (Version 1.25.1) for running DaDIA.R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:a7b28c559f8f3691262fde929500e6c9b01caf33ad6f4926ee316afda26b214c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# extracted-ion-chromatogram-matching

## Summary

Rescue metabolic features detected in DDA pooled-QC runs but missing from DIA feature tables by re-determining their intensities in DIA raw data via extracted-ion-chromatogram (EIC) matching, as implemented in DaDIA.R.

## When to use

Use when a DDA-detected metabolite feature (from pooled QC samples analyzed in DDA mode) lacks an intensity entry in the feature table of a DIA-analyzed individual biological sample, and its abundance must be recovered from the DIA raw data.

## When NOT to use

- When the feature is already detected in the DIA feature table — the rescue step only targets DDA-unique features missing from DIA runs
- When the input is not paired DDA/DIA mzXML data from the same pooled-QC and biological-sample set, since EIC matching requires both acquisition modes of the same cohort

## Inputs

- mzXML files of DIA data from individual biological samples
- mzXML files of DDA data from pooled QC samples
- number of samples (line 38 of DaDIA.R must agree with the real number of DIA mzXML files)

## Outputs

- rescued_features.csv with feature m/z, retention time, and rescued intensity per sample
- metabolite-intensity table with rescued intensities (or NA for unrescued features)

## How to apply

Run DaDIA.R (R 4.0+, XCMS 3.11.4+, metaMS 1.25.1) on paired DDA/DIA mzXML files, first extracting metabolic features with XCMS centWave from both DDA and DIA runs. Identify DDA-unique features whose intensities are missing in the individual-sample DIA runs and flag them for rescue. For each flagged feature, search the DIA raw data by extracted-ion-chromatogram (EIC) matching against the feature's m/z and retention time; if the EIC yields a valid chromatographic peak, assign the recovered intensity to the metabolite-intensity table, otherwise assign NA. Export the resulting rescued-feature table containing feature m/z, retention time, and rescued intensity per sample. Validate rescued EICs by manual inspection of chromatographic peak shapes.

## Related tools

- **DaDIA.R** (R program that performs the feature-rescue mechanism: flags DDA-unique features missing in DIA runs, re-determines their intensities from DIA raw data via EIC matching, and exports the rescued-feature intensity table) — https://github.com/HuanLab/DaDIA.git
- **XCMS** (Used by DaDIA.R for the peak-picking step (centWave function) that extracts metabolic features from both DDA and DIA files prior to EIC matching) — https://github.com/HuanLab/DaDIA.git
- **R** (Runtime environment (Version 4.0 or above) required to execute DaDIA.R)
- **metaMS** (Required dependency (Version 1.25.1) for running DaDIA.R)

## Examples

```
Rscript DaDIA.R
```

## Evaluation signals

- DaDIA.R runs end-to-end without errors on the demo mzXML files composed of DDA and DIA-(SWATH) data distributed via the GitHub repository
- The rescued_features.csv output exists and contains m/z, retention time, and intensity fields
- Rescued-feature counts should be in a range consistent with the paper's report: an average of 344 rescued features from DDA (about 22% of the 1591 DDA features) in urine RP(+) data
- Manual inspection of rescued EICs should show a distribution around the reported 71% true-positive rate and 17% false-positive rate for chromatographic peak shapes
- Features that cannot be rescued must appear as NA in the metabolite-intensity table rather than being silently dropped

## Limitations

- A 17% false-positive rate was observed when manually inspecting rescued EICs, meaning roughly one in six rescued features may not correspond to a genuine chromatographic peak
- 12% of rescued EICs could not be determined (neither good nor clearly bad peak shapes) by manual inspection
- The number of samples declared in line 38 of DaDIA.R must agree with the real number of DIA mzXML files, or the workflow may fail

## Evidence

- [results] a DDA-unique feature missing from a DIA run has its intensity determined from the DIA raw data and assigned to the metabolite-intensity table; if it cannot be rescued, an NA value is assigned: [evidence span withheld — anchor status: not_found]
