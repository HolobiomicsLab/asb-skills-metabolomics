---
name: metabolomics-parameter-translation
description: Use when Paramounter has measured universal parameters from raw LC-MS
  metabolomics data and you need to map them to software-specific parameters (e.g.,
  mzdiff for XCMS CentWave) before feature extraction.
when_to_use_negative:
- Input is already a software-specific parameter set (e.g., an existing XCMS CentWave
  parameter list) — translation is unnecessary
- Universal parameters have not yet been measured by Paramounter; use the parameter-measurement
  step first
- The target program is not one of the supported feature-extraction programs (XCMS
  CentWave, MS-DIAL, AutoTuner, IPO)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Paramounter
  role: Measures universal parameters from raw LC-MS data and translates them into
    software-specific parameters for data processing
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Language in which Paramounter is implemented; used to run the translation
- name: XCMS CentWave
  role: Target feature-extraction program whose software-specific parameters (e.g.,
    mzdiff) are produced by the translation
- name: MS-DIAL
  role: Target program that benefits from translated parameters, providing a threefold
    increase in extracted metabolites vs. defaults
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:b3ee98d9fc427dd46ac07a314e3304e818a9afe7c2238675cba8069ba8da6190
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# metabolomics-parameter-translation

## Summary

Translates the universal peak-detection parameters measured by Paramounter (mass tolerance, peak height, peak width, instrumental shift) into software-specific settings for each target feature-extraction program, including XCMS CentWave's mzdiff dereplication behavior.

## When to use

Use when Paramounter has measured universal parameters from raw LC-MS metabolomics data and you need to map them to software-specific parameters (e.g., mzdiff for XCMS CentWave) before feature extraction.

## When NOT to use

- Input is already a software-specific parameter set (e.g., an existing XCMS CentWave parameter list) — translation is unnecessary
- Universal parameters have not yet been measured by Paramounter; use the parameter-measurement step first
- The target program is not one of the supported feature-extraction programs (XCMS CentWave, MS-DIAL, AutoTuner, IPO)

## Inputs

- Raw LC-MS-based metabolomics data (e.g., mzML/NetCDF files) from which Paramounter measured universal parameters
- Universal parameters output from Paramounter: mass tolerance, peak height, peak width, instrumental shift
- Target processing program name (e.g., XCMS CentWave, MS-DIAL)

## Outputs

- Software-specific parameter set per target program (e.g., XCMS CentWave parameters with mzdiff set to 0.001/0.01, or a negative value to disable dereplication)
- Translated parameters ready for feature extraction

## How to apply

Take the universal parameters Paramounter measured from raw LC-MS data — mass tolerance, peak height, peak width, and instrumental shift — and translate each into the corresponding software-specific parameters of the target processing program. For XCMS CentWave, map the mass tolerance to mzdiff, which serves as the mass tolerance used to dereplicate features with similar m/z values and retention times. The suggested default mzdiff values are 0.001 or 0.01, but because true positive metabolic features with mass differences smaller than that value may be removed by mistake, set mzdiff to any negative value to disable dereplication. Also account for the tuned peak height: Paramounter optimizes it to maximize true positive features, but users can instead apply a higher peak height threshold (e.g., 2X the optimized value) to reduce false positives and avoid software crashes. Write the translated, program-specific parameter set and proceed to feature extraction with the chosen tool.

## Related tools

- **Paramounter** (Measures universal parameters from raw LC-MS data and translates them into software-specific parameters for data processing) — https://github.com/HuanLab/Paramounter
- **R** (Language in which Paramounter is implemented; used to run the translation)
- **XCMS CentWave** (Target feature-extraction program whose software-specific parameters (e.g., mzdiff) are produced by the translation)
- **MS-DIAL** (Target program that benefits from translated parameters, providing a threefold increase in extracted metabolites vs. defaults)

## Examples

```
Rscript paramounter_translate.R --input universal_params.csv --target XCMS_CentWave --output translated_params.csv
```

## Evaluation signals

- The translated parameter set contains the expected software-specific fields (e.g., mzdiff present with a value of 0.001, 0.01, or a negative value when dereplication is disabled)
- Mass-tolerance translation matches the universal mass tolerance measured by Paramounter from the raw LC-MS data
- Peak height threshold falls at or above the Paramounter-optimized value (e.g., 2X the optimized peak height when reducing false positives)
- Downstream feature extraction with translated parameters yields more true positive features than default parameters (e.g., threefold increase in MS-DIAL; 3.7- and 1.6-fold more true positive features than AutoTuner and IPO)
- mzdiff is set to a negative value exactly when the user intends to disable dereplication of similar m/z and retention-time features

## Limitations

- Some true positive metabolic features with mass differences smaller than the suggested mzdiff default (0.001 or 0.01) may be removed by mistake during dereplication
- Using the Paramounter-optimized peak height threshold raises the rate of false positive features and increases the likelihood of software crash
- Paramounter tunes peak height to maximize true positives, which can be suboptimal for datasets where false positives dominate

## Evidence
