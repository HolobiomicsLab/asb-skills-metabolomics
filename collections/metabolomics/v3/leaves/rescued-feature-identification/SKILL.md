---
name: rescued-feature-identification
description: Use when quantifying how many features from a DDA feature list are recovered
  in DaDIA combined data and when validating rescued features by EIC peak-shape inspection
  before trusting rescue metrics.
when_to_use_negative:
- When the analysis goal is full metabolite annotation (MS2 spectral matching against
  MoNA) rather than auditing which DDA features are recovered — use the DaDIA.R annotation
  pipeline instead.
- When the input is already a curated feature table with confirmed identifications
  and no DDA/DIA cross-referencing is needed.
- When data were acquired purely in DIA mode without any DDA runs, since there is
  no DDA feature list to rescue from.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: DaDIA.R
  role: R program that extracts metabolic features from combined DDA/DIA (DaDIA) data
    so that DDA features rescued in the merged feature list can be identified and
    manually EIC-inspected
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Feature-detection engine used by DaDIA.R (version 3.11.4 or higher) to produce
    the DDA and DaDIA feature lists compared during rescue identification
- name: R
  role: Runtime environment (R 4.0 or above) in which DaDIA.R, XCMS, and metaMS are
    executed for the rescue analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:6443420828e7a565f6b62522c8d34b0c3d9529c7d1d6aa98bd30f36959aefef7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rescued-feature-identification

## Summary

Identifies metabolic features from DDA data that are rescued in the combined DaDIA (DDA+DIA) feature list, then verifies each rescued feature by manual extracted-ion chromatogram (EIC) inspection to estimate true- and false-positive rates.

## When to use

Use when quantifying how many features from a DDA feature list are recovered in DaDIA combined data and when validating rescued features by EIC peak-shape inspection before trusting rescue metrics.

## When NOT to use

- When the analysis goal is full metabolite annotation (MS2 spectral matching against MoNA) rather than auditing which DDA features are recovered — use the DaDIA.R annotation pipeline instead.
- When the input is already a curated feature table with confirmed identifications and no DDA/DIA cross-referencing is needed.
- When data were acquired purely in DIA mode without any DDA runs, since there is no DDA feature list to rescue from.

## Inputs

- DDA mzXML files (e.g. pooled QC samples)
- DIA-(SWATH) mzXML files (e.g. individual biological samples)
- DaDIA feature list output (metabolic feature table from DaDIA.R/XCMS)
- Raw chromatographic data (for EIC extraction)

## Outputs

- Count of rescued features from DDA (average per sample)
- True-positive rate of rescued features (EICs with good chromatographic peak shapes)
- False-positive rate of rescued features (EICs with poor/noisy peak shapes)
- Percentage of undetermined rescued features (peak shape not assessable)
- JSON/record of rescue metrics (e.g. urine_rescue_metrics.json)

## How to apply

Load the DDA and DIA-(SWATH) mzXML demo files (e.g. from the DaDIA GitHub repo) into R and run DaDIA.R under XCMS version 3.11.4 or higher to extract metabolic features from both modes and the merged DaDIA result. Cross-reference the DDA feature list against the combined DaDIA feature list and count features present in DDA that are recovered in DaDIA, recording the average number rescued per sample (the urine study reported 344 rescued features, ~22% of the 1591 DDA features). For each rescued feature, extract the EIC from the raw DIA data and manually judge the chromatographic peak shape as good (true positive), poor/noisy (false positive), or undetermined; the urine study's rescue pool yielded 71% true positives, 17% false positives, and 12% undetermined. Compile these counts into a rescue-metrics record (rescued features, true-positive rate, false-positive rate) and verify the numbers match reported figures before downstream use.

## Related tools

- **DaDIA.R** (R program that extracts metabolic features from combined DDA/DIA (DaDIA) data so that DDA features rescued in the merged feature list can be identified and manually EIC-inspected) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Feature-detection engine used by DaDIA.R (version 3.11.4 or higher) to produce the DDA and DaDIA feature lists compared during rescue identification)
- **R** (Runtime environment (R 4.0 or above) in which DaDIA.R, XCMS, and metaMS are executed for the rescue analysis)

## Examples

```
Rscript DaDIA.R  # run after confirming the sample count at line 38 matches the number of DIA mzXML files (Xcms >= 3.11.4)
```

## Evaluation signals

- Rescued-feature count matches the reported average (344 rescued features in the urine study, ~22% of the 1591 DDA features)
- True-positive rate of rescued features equals 71% after manual EIC inspection
- False-positive rate equals 17% (with the remaining 12% undetermined); the three categories sum to 100% of inspected rescued features
- Every field of the rescue-metrics record (e.g. urine_rescue_metrics.json) agrees with the article-reported figures
- The number of samples declared in line 38 of the DaDIA.R script matches the actual number of DIA mzXML files

## Limitations

- A substantial fraction of rescued features can be undetermined by EIC inspection (12% in the urine study), so rescue metrics carry an irreducible ambiguity
- Feature rescue depends on the DaDIA.R merge logic and requires DDA runs on pooled QC samples; rescue cannot be assessed when no DDA feature list exists
- XCMS version dependency: DaDIA.R requires XCMS Development Version 3.11.4 or above (plus R 4.0+ and metaMS 1.25.1), and the sample count in the script must agree with the real number of DIA mzXML files

## Evidence

- [results] Manual EIC inspection of rescued features: 71% true-positive rate, 17% false-positive rate, 12% undetermined: [evidence span withheld — anchor status: not_found]
