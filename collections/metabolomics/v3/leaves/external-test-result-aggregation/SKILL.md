---
name: external-test-result-aggregation
description: Use when evaluating a metabolite identification method with both an internal
  LOOCV evaluation (e.g. 662 KEGG compounds) and an independent external test set
  (e.g. male infertility biomarkers), and you need to tabulate identification validity
  against database-coverage conditions.
when_to_use_negative:
- Input is already an aggregated contingency table or a published validity summary
  rather than per-case identification results
- The goal is raw spectral processing or peak picking rather than post-hoc evaluation
  of identification outcomes
- The unknown compound has no structurally related metabolite in the database, since
  the paper's validity finding does not apply in that regime
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: DeepMASS
  role: Produces the LOOCV and external-test structural-similarity identification
    results that are aggregated and cross-tabulated
  repo: https://github.com/hcji/DeepMASS
- name: KEGG
  role: Reference database of 662 compounds providing known MS/MS spectra and structure-related
    metabolites
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that generated external-test MS/MS spectra of male infertility
    biomarkers
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument that generated external-test MS/MS spectra of male infertility
    biomarkers
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:fc35592c6156b2e2c8d07533064dacd77172405dd52780e5f1279ccbae60813c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# external-test-result-aggregation

## Summary

Aggregates identification results from leave-one-out cross-validation (LOOCV) and an external test set to determine whether DeepMASS identification of an unknown compound is valid when a structure-related metabolite is present in the reference database (KEGG, 662 compounds).

## When to use

Use when evaluating a metabolite identification method with both an internal LOOCV evaluation (e.g. 662 KEGG compounds) and an independent external test set (e.g. male infertility biomarkers), and you need to tabulate identification validity against database-coverage conditions.

## When NOT to use

- Input is already an aggregated contingency table or a published validity summary rather than per-case identification results
- The goal is raw spectral processing or peak picking rather than post-hoc evaluation of identification outcomes
- The unknown compound has no structurally related metabolite in the database, since the paper's validity finding does not apply in that regime

## Inputs

- LOOCV identification results from DeepMASS evaluation on MS/MS spectra of 662 KEGG compounds
- External-test identification results for male infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF
- Reference database entries (KEGG compounds) with structure-relatedness information for each unknown

## Outputs

- Contingency table of identification validity (valid/invalid) vs structure-related database metabolite presence (present/absent)
- Per-case validity labels for LOOCV and external-test identifications
- Assessment of whether results support the finding that identification is valid when a structure-related metabolite is in the database

## How to apply

Load the LOOCV identification results from DeepMASS evaluation on the MS/MS spectra of 662 KEGG compounds, then load the external-test identification results for the male infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF. For each LOOCV and external-test case, determine whether a structure-related metabolite is present in the reference database, then cross-tabulate identification validity (valid/invalid) against structure-related database metabolite presence (present/absent) to build a contingency table. Compare the 'present & valid' and 'absent & invalid' cell counts against the paper's reported finding that identification of an unknown compound is valid if its structure-related metabolite is available in the database. The rationale is that structurally related metabolites often have related MS/MS spectra, so DeepMASS's deep-neural-network structural-similarity scoring can identify unknowns through known ones, extending the range of existing MS/MS databases.

## Related tools

- **DeepMASS** (Produces the LOOCV and external-test structural-similarity identification results that are aggregated and cross-tabulated) — https://github.com/hcji/DeepMASS
- **KEGG** (Reference database of 662 compounds providing known MS/MS spectra and structure-related metabolites)
- **Shimadzu LC-ESI-IT-TOF** (Instrument that generated external-test MS/MS spectra of male infertility biomarkers)
- **Bruker Compact LC-ESI-QTOF** (Instrument that generated external-test MS/MS spectra of male infertility biomarkers)

## Examples

```
python example.py  # run DeepMASS identification on spectra in data/spectra, then aggregate LOOCV and external-test results into a validity x database-coverage contingency table
```

## Evaluation signals

- Contingency table contains four cells (present/absent x valid/invalid) covering every LOOCV and external-test case with no cases dropped
- The 'present & valid' and 'absent & invalid' cells dominate, reproducing the paper's finding that identification is valid when a structure-related metabolite is in the database
- External-test results are restricted to the male infertility biomarkers from the Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF measurements
- LOOCV counts correspond one-to-one with the 662 KEGG compounds evaluated
- Validity labels are consistent across LOOCV and external-test aggregation so both sets can be compared in a single table

## Limitations

- The experimental MetDNA dataset underlying the spectral files was removed from the DeepMASS repository (per README note 2019/04/29), so the package cannot be run directly and the dataset may only be used to reproduce the paper's results
- Validity holds only when a structure-related metabolite is available in the database; identifications without such a related entry are not supported by the paper's finding
- The model used for identification (model-40V) was trained on a specific in-house database; retraining on other spectra may change validity outcomes

## Evidence

- [readme] Take example.py as an example for metabolite identification.: [evidence span withheld — anchor status: external_doc]
- [readme] the dataset has been removed. If you have already download the dataset, please keep it private, and the dataset can be only used to reproduce the results of DeepMASS paper: [evidence span withheld — anchor status: external_doc]
