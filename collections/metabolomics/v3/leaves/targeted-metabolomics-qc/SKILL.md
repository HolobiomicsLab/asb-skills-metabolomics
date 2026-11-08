---
name: targeted-metabolomics-qc
description: Use when processing targeted or semi-targeted CE-, GC-, LC-MS(/MS) or
  HPLC metabolomics experiments (mzML acquisitions plus transition/sequence CSVs)
  where peak picking, calibration, feature QC and quantitation should be automated.
when_to_use_negative:
- Input is already a curated feature table or quantified matrix with no raw chromatograms
  — the skill assumes mzML peak picking through report writing.
- Untargeted full-scan DDA/DIA discovery workflows without a defined transition list
  or internal-standard scheme (use the FIA-MS/DDA BFAIR notebooks instead).
- When a vendor-specific proprietary integration algorithm is required and cannot
  be replaced by SmartPeak's OpenMS-based Gaussian-Mixtures methods.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: SmartPeak
  role: Main application implementing the automated QC and quantitation workflow (peak
    picking, calibration, IS scoring, report generation) on OpenMS
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit providing the MS/MS processing algorithms wrapped by SmartPeak
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: pyOpenMS
  role: Python library for parsing and post-processing the exported quantitation outputs
    (e.g. mzTab-like FeatureDB tables)
  repo: https://pyopenms.readthedocs.io/en/latest/id_by_mz.html?highlight=mztab
- name: BFAIR
  role: Companion package offering targeted-metabolomics analysis and input preparation
    for the SmartPeak MFA workflow
  repo: https://github.com/AutoFlowResearch/BFAIR
- name: Docker
  role: Containerized execution of smartpeak-cli / smartpeak-server for reproducible,
    HPC-scale runs
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:7e113cb4bce2287a8b2b7c501d38b35a1cd712cc4ab15067b68dcb1ffe94ad89
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - BFAIR
  - Docker
  - OpenMS
  - SmartPeak
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# targeted-metabolomics-qc

## Summary

Automated quality-control and quantitation of targeted metabolomics data using SmartPeak on top of OpenMS, covering peak picking, calibration-curve optimization, feature QC, and report generation.

## When to use

Use when processing targeted or semi-targeted CE-, GC-, LC-MS(/MS) or HPLC metabolomics experiments (mzML acquisitions plus transition/sequence CSVs) where peak picking, calibration, feature QC and quantitation should be automated.

## When NOT to use

- Input is already a curated feature table or quantified matrix with no raw chromatograms — the skill assumes mzML peak picking through report writing.
- Untargeted full-scan DDA/DIA discovery workflows without a defined transition list or internal-standard scheme (use the FIA-MS/DDA BFAIR notebooks instead).
- When a vendor-specific proprietary integration algorithm is required and cannot be replaced by SmartPeak's OpenMS-based Gaussian-Mixtures methods.

## Inputs

- mzML raw MS/MS data files (CE-, GC- or LC-MS/MS, HPLC)
- Transition list CSV (MRM components and transitions)
- Sequence/featureDB CSV with sample types and meta-values such as peak_area and logSN
- Calibration and quant-method configuration CSVs

## Outputs

- Selected features with background-interference-filtered peak areas and logSN meta-values
- Calibration curve fits per analyte from calibrators
- Internal-standard score table (is_scores.csv) with per-compound tailing, asymmetry, and height-ratio metrics
- Final quantitation report (CSV) with accuracy, precision, recovery, blank, solvent, RT and feature QC columns

## How to apply

Initialize the workflow by validating acquisition meta-values of transitions, components, component groups, feature filters, and feature QC entries before any processing. Run the SmartPeak pipeline steps in order: LOAD_RAW_DATA and EXTRACT_SPECTRA_WINDOWS, PICK_MRM_FEATURES, SELECT_FEATURES, ESTIMATE and STORE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES, then calibration and quantitation steps, finishing with report writing. Use SmartPeak's Gaussian-Mixtures peak picking and calibration methods rather than generic linear vendor methods, since they outperformed linear methods in targeted proteomics. Invoke validation and IS scoring checks (VALIDATE_IS, COMPUTE_IS_METRICS) via the CLI, GUI, or Dockerized server for HPC workflows, and review intermediate tables (features, calibrators, quant method) in the GUI before final report export.

## Related tools

- **SmartPeak** (Main application implementing the automated QC and quantitation workflow (peak picking, calibration, IS scoring, report generation) on OpenMS) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit providing the MS/MS processing algorithms wrapped by SmartPeak) — https://github.com/AutoFlowResearch/SmartPeak
- **pyOpenMS** (Python library for parsing and post-processing the exported quantitation outputs (e.g. mzTab-like FeatureDB tables)) — https://pyopenms.readthedocs.io/en/latest/id_by_mz.html?highlight=mztab
- **BFAIR** (Companion package offering targeted-metabolomics analysis and input preparation for the SmartPeak MFA workflow) — https://github.com/AutoFlowResearch/BFAIR
- **Docker** (Containerized execution of smartpeak-cli / smartpeak-server for reproducible, HPC-scale runs) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash
```

## Evaluation signals

- 94.3–94.9% correct feature integration was achieved on Spike-in, yeast, mouse-brain and serum datasets; a run well below this range suggests mis-tuned peak picking.
- The workflow completes all steps from LOAD_RAW_DATA through report writing without VALIDATE_* integrity-check errors reported in View|Info.
- Gaussian-Mixtures peak picking/calibration results recover more correct integrations than the generic linear vendor methods on the same transition set.
- Every MRM analyte row has a paired internal standard and non-null IS score (tailing, asymmetry, height ratio) in is_scores.csv.
- Feature integration rate exceeds ~95% when user-defined optimization methods are applied.

## Limitations

- Default feature and chromatogram GUI views are capped at 5000 and 9000 points respectively, limiting interactive review of very large sequences.
- Estimated processing time shown in the GUI is only a rough estimate and is updated only periodically.
- Only about 5% of features remain mis-integrated even with user-defined optimization, typically due to inaccurate peak picking or annotation.
- Current OpenMS versions do not natively support aggregating standards, finding peaks, aligning, applying QC, or writing reports — these are implemented inside SmartPeak, so a change in SmartPeak is needed to alter those steps.

## Evidence

- [results] SmartPeak automated reproducible quantitation and supported all required steps for targeted metabolomics, including peak picking, calibration, pooling, accuracy, precision, recovery, blank, solvent, RT, feature selection, aggregate and feature quality control, quantitation, and report writing: [evidence span withheld — anchor status: not_found]
- [results] Non-linear Gaussian-Mixtures peak picking, calibration, and integrated accuracy and precision algorithmic methods implemented in SmartPeak outperformed generic linear methods in targeted proteomics.: [evidence span withheld — anchor status: not_found]
- [methods] The workflow is initialized by validating the acquisition meta-values of transitions which involves: [evidence span withheld — anchor status: not_found]
- [methods] An optional internal standard scoring step is included: [evidence span withheld — anchor status: not_found]
