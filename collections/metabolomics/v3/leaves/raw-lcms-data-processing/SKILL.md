---
name: raw-lcms-data-processing
description: Use when raw LC-MS metabolomics files must be converted into extracted
  metabolic feature lists and software-specific parameters (mzdiff, peak height, peak
  width) need to be measured from the data itself rather than left at vendor defaults.
when_to_use_negative:
- The input is already an extracted feature table or peak list—Paramounter measures
  parameters from raw LC-MS data prior to feature extraction, so it is not a post-extraction
  refinement step.
- Only one software's defaults need to be reproduced without any parameter measurement
  or benchmarking; the skill's purpose is white-box parameter determination, not blind
  default-parameter runs.
- True positive features with mass differences smaller than the suggested mzdiff defaults
  (0.001 or 0.01) must be preserved and dereplication is unwanted—use the negative
  mzdiff escape only after measuring parameters, and do not rely on the default dereplication
  setting.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters from raw LC-MS
    data and translates them into software-specific parameters for feature extraction
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: Feature extraction program whose default parameters are compared against Paramounter-derived
    settings to quantify the increase in extracted metabolites
- name: XCMS CentWave
  role: Feature extraction program whose mzdiff dereplication parameter and peak-height
    threshold are set from Paramounter-measured universal parameters
- name: AutoTuner
  role: Benchmark tool used to compare true-positive feature counts against Paramounter's
    output
- name: IPO
  role: Benchmark tool used to compare true-positive feature counts against Paramounter's
    output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:1ef48873e1bbf34a1ed8ba1ab38ce1dd05ebb2fa5a1e5bad26eff737ef5d80db
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# raw-lcms-data-processing

## Summary

Directly measure universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS metabolomics data with Paramounter and translate them into software-specific settings for programs such as MS-DIAL and XCMS CentWave, replacing opaque default-parameter processing with a 'white box' approach.

## When to use

Use when raw LC-MS metabolomics files must be converted into extracted metabolic feature lists and software-specific parameters (mzdiff, peak height, peak width) need to be measured from the data itself rather than left at vendor defaults.

## When NOT to use

- The input is already an extracted feature table or peak list—Paramounter measures parameters from raw LC-MS data prior to feature extraction, so it is not a post-extraction refinement step.
- Only one software's defaults need to be reproduced without any parameter measurement or benchmarking; the skill's purpose is white-box parameter determination, not blind default-parameter runs.
- True positive features with mass differences smaller than the suggested mzdiff defaults (0.001 or 0.01) must be preserved and dereplication is unwanted—use the negative mzdiff escape only after measuring parameters, and do not rely on the default dereplication setting.

## Inputs

- Raw LC-MS-based metabolomics data files (e.g., LC-MS comparison datasets referenced in the article's MS-DIAL comparison)
- Universal parameter targets: mass tolerance, peak height, peak width, instrumental shift

## Outputs

- Measured universal parameters (mass tolerance, peak height, peak width, instrumental shift) derived from the raw data
- Software-specific parameter sets (e.g., cond_paramounter_msdial for MS-DIAL, mzdiff/peak-height settings for XCMS CentWave)
- Extracted metabolic feature lists (metabolite counts) from programs such as MS-DIAL or XCMS CentWave

## How to apply

Run Paramounter (an R program available from HuanLab/Paramounter) on the raw LC-MS files to automatically measure the universal parameters—mass tolerance, peak height, peak width, and instrumental shift—from the data prior to metabolic feature extraction. Paramounter then translates these universal parameters into software-specific parameters for downstream programs: for XCMS CentWave, mass tolerance maps to the mzdiff dereplication parameter (suggested defaults 0.001 or 0.01; set mzdiff to any negative value to disable dereplication when true positive features with smaller mass differences must be retained). For peak height, Paramounter tunes an optimized threshold that maximizes true positive features; because this optimized value raises the false-positive rate and can crash the software, users may raise the threshold (e.g., 2X the optimized peak height) to reduce false positives. For MS-DIAL-based feature extraction, feed the Paramounter-derived settings (cond_paramounter_msdial) and run the same raw data also under MS-DIAL defaults (cond_default_msdial) to compare extracted-feature counts—expect roughly a threefold increase in extracted metabolites versus defaults. Validate the pipeline further by benchmarking true-positive feature counts against AutoTuner and IPO, where Paramounter should yield about 3.7-fold and 1.6-fold more true positives respectively, and confirm robustness across the 11 validation datasets covering different sample types, acquisition modes, and MS vendors.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters from raw LC-MS data and translates them into software-specific parameters for feature extraction) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (Feature extraction program whose default parameters are compared against Paramounter-derived settings to quantify the increase in extracted metabolites)
- **XCMS CentWave** (Feature extraction program whose mzdiff dereplication parameter and peak-height threshold are set from Paramounter-measured universal parameters)
- **AutoTuner** (Benchmark tool used to compare true-positive feature counts against Paramounter's output)
- **IPO** (Benchmark tool used to compare true-positive feature counts against Paramounter's output)

## Examples

```
Rscript paramounter_run.R --input raw_lcms/ --output cond_paramounter_msdial/
```

## Evaluation signals

- Running MS-DIAL with Paramounter-derived parameters (cond_paramounter_msdial) on the same raw LC-MS files should produce approximately a threefold increase in extracted metabolites compared to MS-DIAL defaults (cond_default_msdial).
- Comparing true-positive feature counts against AutoTuner should show Paramounter generating about 3.7-fold more true positive features, and about 1.6-fold more than IPO.
- mzdiff set to a negative value must disable dereplication entirely, retaining features with similar m/z and retention times that would otherwise be removed by the default 0.001/0.01 mass tolerance.
- Peak-height threshold raised to 2X the Paramounter-optimized value should visibly reduce the number of false positive features relative to the optimized threshold.
- The measured universal parameters (mass tolerance, peak height, peak width, instrumental shift) must be consistent across the 11 validation datasets covering different sample types, data acquisition modes, and MS vendors.

## Limitations

- The Paramounter-optimized peak height maximizes true positive features but simultaneously raises the rate of false positive features and increases the likelihood of software crash.
- Suggested mzdiff defaults (0.001 or 0.01) can mistakenly remove true positive metabolic features whose mass differences fall below that value.
- No discussion-section content was available, so further limitations, failure modes, or reproducibility caveats could not be extracted.

## Evidence
