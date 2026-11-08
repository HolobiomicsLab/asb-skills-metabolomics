---
name: lcms-feature-extraction-parameterization
description: Use when raw LC-MS-based metabolomics data must be processed for feature
  extraction and default software parameters underperform, or when you need data-driven
  values for mass tolerance, peak height, peak width, or instrumental shift.
when_to_use_negative:
- Input is already a processed feature table or peak list rather than raw LC-MS data
  — Paramounter measures universal parameters from raw instrument spectra.
- Only a parameter for a program with no documented universal-parameter mapping is
  needed; the translation layer in Paramounter targets software-specific parameters
  of known feature-extraction programs.
- The analysis goal is downstream statistical analysis of existing features, not parameterization
  of feature extraction.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: Paramounter
  role: Measures universal parameters (mass tolerance, peak height, peak width, instrumental
    shift) from raw LC-MS data and translates them into software-specific parameters
    for feature extraction programs
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Programming environment in which Paramounter runs
- name: XCMS CentWave
  role: Target feature-extraction program whose mzdiff mass tolerance and peak height
    threshold are set from translated universal parameters
- name: MS-DIAL
  role: Target feature-extraction program shown to extract threefold more metabolites
    when parameterized with Paramounter versus defaults
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:8540f5a7a4e342d7050fc3998189da6ad41b67d997a7db36c80b84b95d6f0cd4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-feature-extraction-parameterization

## Summary

Automatically measure universal LC-MS metabolomics parameters (mass tolerance, peak height, peak width, instrumental shift) from raw data using Paramounter, then translate them into software-specific settings such as XCMS CentWave's mzdiff to tune feature extraction.

## When to use

Use when raw LC-MS-based metabolomics data must be processed for feature extraction and default software parameters underperform, or when you need data-driven values for mass tolerance, peak height, peak width, or instrumental shift.

## When NOT to use

- Input is already a processed feature table or peak list rather than raw LC-MS data — Paramounter measures universal parameters from raw instrument spectra.
- Only a parameter for a program with no documented universal-parameter mapping is needed; the translation layer in Paramounter targets software-specific parameters of known feature-extraction programs.
- The analysis goal is downstream statistical analysis of existing features, not parameterization of feature extraction.

## Inputs

- Raw LC-MS-based metabolomics data files (instrument spectra) prior to metabolic feature extraction
- Target feature-extraction program choice (e.g., XCMS CentWave, MS-DIAL)

## Outputs

- Measured universal parameters: mass tolerance, peak height, peak width, instrumental shift
- Translated software-specific parameters for each target program (e.g., mzdiff for XCMS CentWave)
- Improved feature extraction results compared to default settings

## How to apply

Launch an R session, load Paramounter from HuanLab/Paramounter, and point it at the raw LC-MS data file so it can read the instrument spectra. Paramounter then automatically measures the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from the raw data prior to metabolic feature extraction, using rank-based intensity sorting and zone of interest concepts. Next, translate each universal parameter into its software-specific counterpart (e.g., for XCMS CentWave, mzdiff acts as the mass tolerance used to dereplicate features with similar m/z values and retention times; suggested defaults are 0.001 or 0.01). If dereplication should be disabled, set mzdiff to any negative value, because true positive features with mass differences smaller than the default can otherwise be removed by mistake. For XCMS CentWave peak height, use the optimized value that maximizes true positive features, but consider raising it (e.g., 2X the optimized threshold) when false positives or software crashes become a problem.

## Related tools

- **Paramounter** (Measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS data and translates them into software-specific parameters for feature extraction programs) — https://github.com/HuanLab/Paramounter
- **R** (Programming environment in which Paramounter runs)
- **XCMS CentWave** (Target feature-extraction program whose mzdiff mass tolerance and peak height threshold are set from translated universal parameters)
- **MS-DIAL** (Target feature-extraction program shown to extract threefold more metabolites when parameterized with Paramounter versus defaults)

## Examples

```
Rscript -e 'library(Paramounter); params <- Paramounter::measure_universal_params(raw_lcms_data); print(params)'
```

## Evaluation signals

- Output parameter record contains fields for all four universal parameters (mass tolerance, peak height, peak width, instrumental shift) with plausible numeric values
- Translated mzdiff for XCMS CentWave lies within the suggested default range (0.001–0.01) unless dereplication was deliberately disabled via a negative value
- Feature extraction with parameterized settings yields more true positive features than default-parameter runs (Paramounter gave 3.7-fold vs AutoTuner and 1.6-fold vs IPO)
- Applied peak height threshold (default or 2X raised) reduces false positives without crashing the software

## Limitations

- The optimized peak height that maximizes true positive features also raises the false positive rate and increases the likelihood of software crash; users must manually raise the threshold (e.g., 2X) to trade off
- True positive metabolic features with mass differences smaller than the suggested mzdiff default (0.001 or 0.01) may be removed by mistake during dereplication
- No discussion-section content was provided, so broader limitations or reproducibility gaps could not be extracted

## Evidence
