---
name: internal-standard-response-normalization
description: Use when processing targeted LC-MS(/MS), CE-, GC-MS, HPLC, or FIA-MS
  injections where each transition or component is paired with an internal standard
  and per-feature peak areas must be ratio-normalized before calibration or quantitation.
when_to_use_negative:
- Data are untargeted full-scan or DDA/DIA acquisitions with no assigned internal-standard
  transitions — use untargeted FIA-MS post-processing (e.g. BFAIR) instead
- The input is already a normalized feature table or concentration report; re-normalizing
  would double-correct for instrument drift
- The experiment is flux analysis where labeling fractions, not area ratios, are the
  response (Targeted_flux_analysis workflows)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SmartPeak
  role: Executes the automated workflow including the optional internal standard scoring
    step and computes ratio-normalized responses for calibration and quantitation
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: pyOpenMS
  role: Parses the FeatureDB report to extract per-feature peak_area metavalues needed
    to compute analyte/IS ratios in Python
- name: OpenMS
  role: Toolkit underlying SmartPeak's feature extraction and calibration steps on
    which ratio normalization is built
- name: SmartPeak GUI
  role: Used to inspect calibration fitted model and internal-standard assignment
    via the Calibrators and Quant Methods views
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:885edf4463b2335ce8fb9285425a1b9f604ca26ab155339d6709802a06d5d282
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - SmartPeak GUI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# internal-standard-response-normalization

## Summary

Normalizes targeted metabolomics responses by computing analyte-to-internal-standard peak-area ratios per calibration level and per sample, so that instrument drift and injection variability cancel before calibration-curve fitting or quantitation in SmartPeak/OpenMS workflows.

## When to use

Use when processing targeted LC-MS(/MS), CE-, GC-MS, HPLC, or FIA-MS injections where each transition or component is paired with an internal standard and per-feature peak areas must be ratio-normalized before calibration or quantitation.

## When NOT to use

- Data are untargeted full-scan or DDA/DIA acquisitions with no assigned internal-standard transitions — use untargeted FIA-MS post-processing (e.g. BFAIR) instead
- The input is already a normalized feature table or concentration report; re-normalizing would double-correct for instrument drift
- The experiment is flux analysis where labeling fractions, not area ratios, are the response (Targeted_flux_analysis workflows)

## Inputs

- Sequence of injections (calibration standards, QCs, blanks, solvents, unknowns) in .mzML format from CE-, GC-, LC-MS(/MS) or HPLC instruments
- Targeted feature report (FeatureDB / pyOpenMS-parsed) with per-feature metavalues including peak_area for each transition and component
- Transition list assigning internal standards to analyte components (SmartPeak transition table)
- SmartPeak session configuration specifying sample types and the workflow step list (e.g. VALIDATE_IS, COMPUTE_IS_METRICS)

## Outputs

- Per-injection, per-transition analyte-to-IS peak-area response ratios
- IS-normalized calibration points feeding the Gaussian-Mixtures calibration fit
- Back-calculated concentrations and QC metrics (accuracy, precision, recovery, blank, solvent) computed on normalized responses
- SmartPeak report export with normalized response and quantitation columns

## How to apply

Load the injected sequence (e.g. the Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP tutorial dataset) and read per-feature metadata such as peak_area from the FeatureDB-style report using SmartPeak or pyOpenMS. For each analyte transition, locate its assigned internal standard (SmartPeak supports an optional internal standard scoring step, VALIDATE_IS / COMPUTE_IS_METRICS) and compute the ratio of analyte peak area to internal-standard peak area for every calibration standard, QC, and unknown injection. Use these response ratios — not raw areas — as the response variable for fitting the non-linear Gaussian-Mixtures calibration curve, and apply the same ratio normalization when back-calculating concentrations and computing accuracy, precision, recovery, blank, and solvent QC metrics. The rationale is that ratiometric responses absorb run-to-run detector and injection variability, which is why SmartPeak's integrated accuracy and precision algorithms outperformed generic linear vendor methods; validate by confirming the fitted calibration achieves R-squared near the reported 0.99 and that the workflow retains >95% correctly integrated features after user-defined optimization.

## Related tools

- **SmartPeak** (Executes the automated workflow including the optional internal standard scoring step and computes ratio-normalized responses for calibration and quantitation) — https://github.com/AutoFlowResearch/SmartPeak
- **pyOpenMS** (Parses the FeatureDB report to extract per-feature peak_area metavalues needed to compute analyte/IS ratios in Python)
- **OpenMS** (Toolkit underlying SmartPeak's feature extraction and calibration steps on which ratio normalization is built)
- **SmartPeak GUI** (Used to inspect calibration fitted model and internal-standard assignment via the Calibrators and Quant Methods views)

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash   # then run SmartPeak CLI workflow with VALIDATE_IS / COMPUTE_IS_METRICS steps
```

## Evaluation signals

- Fitted Gaussian-Mixtures calibration on IS-normalized ratios reaches R-squared ≈ 0.99 (accept 0.98–1.00)
- Automated workflow correctly integrates ≥ 88–95% of features (e.g. 94.3% of 574 Spike-in features); user-optimized workflows exceed 95%
- Every analyte transition has a non-missing internal-standard assignment before ratio computation; blanks and solvents flag recovery outliers
- Normalized responses absorb run-to-run drift: replicate calibration injections at the same level show stable ratios whereas raw areas vary
- Report export contains response-ratio and concentration columns consistent with the chosen quantitation method

## Limitations

- Requires correctly annotated internal standards; misassigned or missing IS transitions propagate error into every ratio and calibration point
- Peak picking and integration errors upstream (≈5% of features even after optimization) corrupt area ratios for affected transitions
- The optional internal standard scoring step is not mandatory, so workflows can silently omit IS validation if not cherry-picked in the workflow editor
- Ratio normalization assumes the IS response tracks the analyte under the same acquisition method; it does not correct gross acquisition failures detectable only by feature QC

## Evidence

- [other] Calibration is a relative quantitation technique in which the response relationship between the amount of a metabolite and the instrument detector output is estimated by a calibration curve fit to calibration samples: [evidence span withheld — anchor status: not_found]
- [methods] An optional internal standard scoring step is included: [evidence span withheld — anchor status: not_found]
- [results] SmartPeak automated reproducible quantitation and supported all required steps for targeted metabolomics, including peak picking, calibration, pooling, accuracy, precision, recovery, blank, solvent, RT, feature selection, aggregate and feature quality control, quantitation: [evidence span withheld — anchor status: not_found]
- [results] Non-linear Gaussian-Mixtures peak picking, calibration, and integrated accuracy and precision algorithmic methods implemented in SmartPeak outperformed generic linear methods in targeted proteomics: [evidence span withheld — anchor status: not_found]
