---
name: lcms-raw-data-parameter-measurement
description: Use when starting an untargeted LC-MS metabolomics workflow with raw
  spectra and you need data-driven values for mass tolerance, peak height, peak width,
  or instrumental shift instead of relying on vendor defaults or manual tuning.
when_to_use_negative:
- Input is already a metabolite/feature table rather than raw LC-MS spectra — Paramounter
  measures parameters from raw data prior to feature extraction.
- You need software-specific parameter recommendations that go beyond the four universal
  parameters (mass tolerance, peak height, peak width, instrumental shift).
- A simpler, non-R toolchain is required; Paramounter is an R program and its GitHub
  distribution is the supported route.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Paramounter
  role: R program that automatically measures the universal parameters from raw LC-MS
    data prior to feature extraction
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Runtime environment in which Paramounter is executed
- name: MS-DIAL
  role: Downstream feature extraction program whose default-parameter output was benchmarked
    against Paramounter-derived parameters
- name: XCMS CentWave
  role: Feature extraction program whose mzdiff parameter corresponds to the measured
    mass tolerance
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:ad4a07dace75a07cd72b63efc6788746155369ae8b64591161bbb18090c0cbf7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lcms-raw-data-parameter-measurement

## Summary

Automatically measure the universal LC-MS feature-extraction parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from raw LC-MS metabolomics data using Paramounter, an R program, before any feature extraction.

## When to use

Use when starting an untargeted LC-MS metabolomics workflow with raw spectra and you need data-driven values for mass tolerance, peak height, peak width, or instrumental shift instead of relying on vendor defaults or manual tuning.

## When NOT to use

- Input is already a metabolite/feature table rather than raw LC-MS spectra — Paramounter measures parameters from raw data prior to feature extraction.
- You need software-specific parameter recommendations that go beyond the four universal parameters (mass tolerance, peak height, peak width, instrumental shift).
- A simpler, non-R toolchain is required; Paramounter is an R program and its GitHub distribution is the supported route.

## Inputs

- Raw LC-MS-based metabolomics data files (instrument spectra)
- R session with the Paramounter package from HuanLab/Paramounter

## Outputs

- Measured universal parameters: mass tolerance, peak height, peak width, instrumental shift
- Software-specific parameter values for downstream feature extraction programs (e.g., MS-DIAL, XCMS CentWave)

## How to apply

Load the Paramounter R package from HuanLab/Paramounter and point it at the raw LC-MS-based metabolomics data so it can read the instrument spectra directly. Run its automatic measurement routine, which uses novel concepts including rank-based intensity sorting and a zone of interest to compute the four universal parameters — mass tolerance, peak height, peak width, and instrumental shift — prior to any metabolic feature extraction. Translate these universal parameters into software-specific parameters for the downstream extraction program (e.g., MS-DIAL or XCMS CentWave), noting that Paramounter's peak height is tuned to maximize the number of true positive features, so a higher threshold (e.g., 2X the optimized value) may be chosen to suppress false positives. Validate the run by confirming that each of the four parameters holds a plausible numeric value appropriate for LC-MS feature extraction.

## Related tools

- **Paramounter** (R program that automatically measures the universal parameters from raw LC-MS data prior to feature extraction) — https://github.com/HuanLab/Paramounter
- **R** (Runtime environment in which Paramounter is executed)
- **MS-DIAL** (Downstream feature extraction program whose default-parameter output was benchmarked against Paramounter-derived parameters)
- **XCMS CentWave** (Feature extraction program whose mzdiff parameter corresponds to the measured mass tolerance)

## Examples

```
Rscript -e 'library(Paramounter); params <- measure_universal_parameters("raw_LCMS.mzML")'
```

## Evaluation signals

- The output parameter record contains fields for all four universal parameters (mass tolerance, peak height, peak width, instrumental shift), each with a plausible numeric value for LC-MS feature extraction.
- Cross-validation against 11 datasets spanning different sample types, acquisition modes, and MS vendors shows the parameters transfer robustly.
- Feature extraction driven by Paramounter-derived parameters yields roughly a threefold increase in extracted metabolites versus MS-DIAL default parameters.
- Paramounter's peak-height tuning produces 3.7-fold and 1.6-fold more true positive features than AutoTuner and IPO, respectively.

## Limitations

- Paramounter tunes an optimized peak height that maximizes true positive features but comes with a higher rate of false positive features and the likelihood of software crash; a higher threshold (e.g., 2X the optimized value) reduces false positives.
- The mzdiff mass tolerance used for dereplication of XCMS CentWave features may remove true positive features whose mass differences fall below the suggested default (0.001 or 0.01); setting mzdiff to a negative value disables dereplication.
- No discussion section content was available, so limitations beyond those noted in the abstract/intro/README could not be extracted.

## Evidence
