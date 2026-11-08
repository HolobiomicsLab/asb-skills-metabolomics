---
name: chromatographic-peak-width-measurement
description: Use when you need to set chromatographic peak-width parameters for an
  LC-MS feature extraction run (e.g. XCMS CentWave peak width, MS-DIAL tolerances)
  from raw untargeted metabolomics data.
when_to_use_negative:
- Your data are already a processed feature table and peak parameters were previously
  fixed — the skill measures parameters from raw LC-MS data, not from extracted features.
- You only need a single non-LC-MS parameter (e.g. alignment or statistics settings)
  — the skill targets the four universal LC-MS parameters measured by Paramounter.
- You require dereplication filtering decisions (mzdiff handling) rather than peak-width
  estimation — that is handled separately in the README notes on mzdiff, not by the
  peak-width measurement itself.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Paramounter
  role: R program that automatically measures the universal parameters, including
    chromatographic peak width, from raw LC-MS data and translates them into software-specific
    parameters
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: runtime environment in which the Paramounter package is executed to perform
    the parameter measurement
- name: MS-DIAL
  role: downstream feature-extraction program whose default-parameter performance
    was benchmarked against Paramounter-derived parameters (threefold more metabolites
    extracted with Paramounter)
- name: XCMS CentWave
  role: feature-extraction algorithm whose peak-detection parameters (e.g. mzdiff
    as mass tolerance) are set from the Paramounter-measured universal values
- name: AutoTuner
  role: benchmark tool against which Paramounter's parameter-driven feature extraction
    was compared (Paramounter yielded 3.7-fold more true positive features)
- name: IPO
  role: benchmark tool against which Paramounter's parameter-driven feature extraction
    was compared (Paramounter yielded 1.6-fold more true positive features)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:3fdd8f6f2145ba9333e6f5f7ec5689484df4a3034087ce28533af9c66b95f750
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - AutoTuner
  - IPO
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chromatographic-peak-width-measurement

## Summary

Paramounter automatically measures the chromatographic peak width from raw LC-MS metabolomics data, prior to feature extraction, as one of four universal parameters used to tune feature-extraction software like XCMS CentWave or MS-DIAL.

## When to use

Use when you need to set chromatographic peak-width parameters for an LC-MS feature extraction run (e.g. XCMS CentWave peak width, MS-DIAL tolerances) from raw untargeted metabolomics data.

## When NOT to use

- Your data are already a processed feature table and peak parameters were previously fixed — the skill measures parameters from raw LC-MS data, not from extracted features.
- You only need a single non-LC-MS parameter (e.g. alignment or statistics settings) — the skill targets the four universal LC-MS parameters measured by Paramounter.
- You require dereplication filtering decisions (mzdiff handling) rather than peak-width estimation — that is handled separately in the README notes on mzdiff, not by the peak-width measurement itself.

## Inputs

- raw LC-MS-based metabolomics data (instrument spectra files) prior to metabolic feature extraction

## Outputs

- measured universal parameters, including chromatographic peak width (alongside mass tolerance, peak height, and instrumental shift)
- software-specific parameter values ready for feature extraction in programs such as XCMS CentWave or MS-DIAL

## How to apply

Load the Paramounter R package from the HuanLab/Paramounter repository and point it at your raw LC-MS spectra so it can measure the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from the instrument data before any feature extraction. Among these, the peak-width measurement uses rank-based intensity sorting and a zone-of-interest concept to characterize chromatographic peak breadth in the raw profiles. Once measured, Paramounter translates the peak width (together with the other universal parameters) into software-specific parameter values for whichever feature extraction program you use, e.g. the peak width handled by XCMS CentWave or the default-parameter MS-DIAL extraction. Note that Paramounter's peak-height tuning can raise false-positive rates and even cause software crashes, so in that dimension users may prefer a threshold like 2X the optimized peak-height value rather than the raw optimum. Run the measurement, inspect the exported parameter record, and feed the resulting software-specific values into your downstream feature extraction pipeline.

## Related tools

- **Paramounter** (R program that automatically measures the universal parameters, including chromatographic peak width, from raw LC-MS data and translates them into software-specific parameters) — https://github.com/HuanLab/Paramounter
- **R** (runtime environment in which the Paramounter package is executed to perform the parameter measurement)
- **MS-DIAL** (downstream feature-extraction program whose default-parameter performance was benchmarked against Paramounter-derived parameters (threefold more metabolites extracted with Paramounter))
- **XCMS CentWave** (feature-extraction algorithm whose peak-detection parameters (e.g. mzdiff as mass tolerance) are set from the Paramounter-measured universal values)
- **AutoTuner** (benchmark tool against which Paramounter's parameter-driven feature extraction was compared (Paramounter yielded 3.7-fold more true positive features))
- **IPO** (benchmark tool against which Paramounter's parameter-driven feature extraction was compared (Paramounter yielded 1.6-fold more true positive features))

## Examples

```
Rscript -e 'library(Paramounter); pm <- paramounter::measure_parameters(raw_lcms); pm$peak_width'
```

## Evaluation signals

- The exported parameter record must contain a numeric field for peak width alongside fields for mass tolerance, peak height, and instrumental shift.
- Measured peak-width values should be plausible for LC-MS chromatography (not zero, not absurdly large) before being translated into CentWave/MS-DIAL parameters.
- When the translated peak-width and peak-height parameters are applied in XCMS CentWave, expect roughly a threefold increase in extracted metabolites versus default MS-DIAL parameters, and 3.7-fold / 1.6-fold more true positive features than AutoTuner / IPO.
- Because the optimized peak height can inflate false positives, check whether raising the peak-height threshold (e.g. 2X the optimized value) reduces false positives without collapsing the feature count.
- Validation across the 11 datasets covering different sample types, acquisition modes, and MS vendors should show that the measurement is robust to changes in instrument vendor and sample matrix.

## Limitations

- The optimized peak-height value yields a higher rate of false positive features and increases the likelihood of software crash; users may need to raise the threshold (e.g. 2X the optimized peak height).
- The mzdiff-based dereplication (mass tolerance) can mistakenly remove true positive features whose mass differences are smaller than the suggested 0.001 or 0.01; a negative mzdiff disables dereplication.
- The paper's discussion section was not available in the provided context, so any deeper limitations on peak-width accuracy were not extractable.

## Evidence

- [abstract] Paramounter automatically measures the universal parameters (mass tolerance, peak height, peak width, instrumental shift) directly from raw LC-MS-based metabolomics data: [evidence span withheld — anchor status: not_found]
- [abstract] Paramounter also translates universal parameters to software-specific parameters for data processing in different programs: "Paramounter also translates universal parameters to software-specific parameters for data processing in different programs"
- [readme] mzdiff is used as the mass tolerance to dereplicate the features (similar m/z values and retention times) extracted by XCMS CentWave.: "mzdiff is used as the mass tolerance to dereplicate the features (similar m/z values and retention times) extracted by XCMS CentWave. Suggested default value: 0.001 or 0.01."
- [readme] users can try a higher peak height threshold to reduce the number of false positive features (e.g., 2X the optimized peak height threshold).: "users can try a higher peak height threshold to reduce the number of false positive features (e.g., 2X the optimized peak height threshold)."
