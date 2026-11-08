---
name: centwave-peak-picking-parameterization
description: Use when raw LC-HRMS data (mzXML from e.g. Bruker Impact II UHR-QqTOF
  runs) must be converted into a feature table, and the analysis relies on XCMS 3.11.4+
  centWave feature detection feeding downstream rescue or annotation steps.
when_to_use_negative:
- The data are already a processed feature table — centWave only operates on raw mzXML
  LC-MS files.
- You need MS2 spectral annotation only, without re-picking features; use the downstream
  dot-product scoring step of DaDIA.R instead.
- Your XCMS version is below 3.11.4 — the article states this minimum is required
  to run DaDIA.R.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Provides the centWave peak-picking function used inside DaDIA.R to extract
    metabolic features from DDA and DIA mzXML data
  repo: https://github.com/HuanLab/DaDIA.git
- name: DaDIA.R
  role: R script that wraps centWave feature extraction and integrates results across
    DDA/DIA files
  repo: https://github.com/HuanLab/DaDIA.git
- name: R
  role: Runtime environment (version 4.0 or above) in which DaDIA.R and centWave are
    executed
- name: metaMS
  role: Required companion R package (version 1.25.1) for the DaDIA.R environment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:7cd0df779cde0cb45c2a849d1a9e896d0592d9ce804e3214dca9fe8a4bc758e7
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# centwave-peak-picking-parameterization

## Summary

Parameterize and run XCMS's centWave peak-picking to extract metabolic features from DaDIA LC-MS mzXML files (DDA pooled-QC and DIA individual samples), as implemented inside the DaDIA.R pipeline.

## When to use

Use when raw LC-HRMS data (mzXML from e.g. Bruker Impact II UHR-QqTOF runs) must be converted into a feature table, and the analysis relies on XCMS 3.11.4+ centWave feature detection feeding downstream rescue or annotation steps.

## When NOT to use

- The data are already a processed feature table — centWave only operates on raw mzXML LC-MS files.
- You need MS2 spectral annotation only, without re-picking features; use the downstream dot-product scoring step of DaDIA.R instead.
- Your XCMS version is below 3.11.4 — the article states this minimum is required to run DaDIA.R.

## Inputs

- mzXML files of DIA runs from individual biological samples (e.g. urine or leukemia plasma LC-MS)
- mzXML file(s) of DDA pooled QC sample runs
- R environment with XCMS >= 3.11.4 and metaMS 1.25.1

## Outputs

- Metabolic feature table (m/z, retention time, intensity) per file from centWave peak-picking
- Integrated DDA/DIA feature set consumed by the DaDIA.R rescue and annotation steps

## How to apply

Run the centWave function from XCMS (Development Version 3.11.4 or above, within R 4.0+ alongside metaMS 1.25.1) on each mzXML file to pick metabolic features from both the DDA pooled-QC run and the DIA runs of individual biological samples. In the DaDIA.R workflow, centWave output is then integrated so DDA-unique features missing from DIA runs can be rescued by EIC matching; because DDA and DIA files are processed together, the same peak-picking parameterization must be applied across file types so m/z and retention time are comparable. The README warns that the number of samples must be set correctly (line 38 must agree with the real number of DIA mzXML files), otherwise the centWave-driven extraction and downstream integration will be misaligned. Validate by running the demo mzXML files from the HuanLab/DaDIA GitHub repository end-to-end without errors before applying to real data.

## Related tools

- **XCMS** (Provides the centWave peak-picking function used inside DaDIA.R to extract metabolic features from DDA and DIA mzXML data) — https://github.com/HuanLab/DaDIA.git
- **DaDIA.R** (R script that wraps centWave feature extraction and integrates results across DDA/DIA files) — https://github.com/HuanLab/DaDIA.git
- **R** (Runtime environment (version 4.0 or above) in which DaDIA.R and centWave are executed)
- **metaMS** (Required companion R package (version 1.25.1) for the DaDIA.R environment)

## Examples

```
Rscript DaDIA.R
```

## Evaluation signals

- DaDIA.R must run end-to-end without errors on the authors' demo mzXML files composed of DDA and DIA-(SWATH) data.
- The exported feature/rescue table must exist and contain m/z, retention time, and intensity fields for each sample.
- The sample count declared in the DaDIA.R script (line 38) matches the actual number of DIA mzXML files.
- Rescued-feature counts fall in a plausible range: the paper reports ~344 rescued features from DDA (≈22% of 1591 DDA features) in urine RP(+) mode.
- Manual EIC inspection of rescued features shows roughly 71% true-positive and 17% false-positive chromatographic peak shapes.

## Limitations

- Requires R 4.0 or above, XCMS 3.11.4 or above, and metaMS 1.25.1; older versions are not supported by DaDIA.R.
- The number of samples hardcoded in DaDIA.R (line 38) must be manually updated to match the real number of DIA mzXML files.
- Rescue of DDA-only features via EIC matching is imperfect — about 17% of manually inspected rescued EICs were false positives and 12% were undetermined.
- Parameterization must be consistent between DDA and DIA files for downstream integration; mismatched peak-picking settings will break feature matching.

## Evidence

- [results] Run DaDIA.R on the demo data, using its peak-picking step based on the centWave function in XCMS to extract metabolic features from both the DDA and DIA files.: [evidence span withheld — anchor status: not_found]
- [readme] R Version 4.0 or above, XCMS Development Version 3.11.4 or above, and metaMS Version 1.25.1 are required: "R Version 4.0 or above, XCMS Development Version 3.11.4 or above, and metaMS Version 1.25.1"
- [methods] XCMS version 3.11.4 or higher is required to run DaDIA.R: "XCMS version 3.11.4 or higher is required to run DaDIA.R"
- [readme] The number of samples in line 38 has to agree with the real number of DIA mzXML files: "The number of samples in line 38 has to agree with the real number of DIA mzXML files"
- [results] flag metabolic features detected in the DDA (pooled-QC) run whose intensities are missing in the individual-sample DIA runs, and re-determine their intensities in the DIA data by extracted-ion-chromatogram (EIC) matching: [evidence span withheld — anchor status: not_found]
