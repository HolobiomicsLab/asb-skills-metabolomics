---
name: manual-annotation-concordance-comparison
description: Use when validating an automated GC-MS processing pipeline (e.g. uafR)
  against published results by reproducing compound counts and statistical patterns
  (ANOSIM/NMDS) from a study that used traditional manual peak annotation.
when_to_use_negative:
- The published study used a different mass-spectrometer modality whose spectra uafR
  cannot process (e.g. MS2, MS3, or ion mobility + MS2 data), so re-analysis would
  not be comparable
- The published dataset is already a feature table or standardized abundance matrix
  rather than a raw vendor hit table, so spreadOut()/mzExacto cannot run
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: R package whose GC-MS workflow (spreadOut, mzExacto, standardifyIt) re-processes
    the published hit table so its output can be compared to the manual peak annotations
  repo: https://github.com/castratton/uafR
- name: Agilent Unknowns Analysis
  role: Vendor deconvolution software that produced the tentative peak identifications
    which both the original manual annotation and the uafR re-analysis build upon
- name: PubChem
  role: Open-source chemical database used by mzExacto decontamination to drop chemicals
    lacking a PubChem entry or canonical SMILES, keeping the re-analysis concordant
    with published compound types
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:0f1e4e95be1eb954fc665b28704c4c20a31cbe078ac3f1e340c1b11eb3413771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - LOTUS
  - PubChem
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# manual-annotation-concordance-comparison

## Summary

A validation skill that re-runs automated GC-MS compound identification and downstream statistics on a previously published dataset and checks whether the compounds identified and their significance patterns are concordant with the original manual peak-annotation workflow.

## When to use

Use when validating an automated GC-MS processing pipeline (e.g. uafR) against published results by reproducing compound counts and statistical patterns (ANOSIM/NMDS) from a study that used traditional manual peak annotation.

## When NOT to use

- The published study used a different mass-spectrometer modality whose spectra uafR cannot process (e.g. MS2, MS3, or ion mobility + MS2 data), so re-analysis would not be comparable
- The published dataset is already a feature table or standardized abundance matrix rather than a raw vendor hit table, so spreadOut()/mzExacto cannot run

## Inputs

- Published GC-MS hit table exported from vendor peak-detection software (CSV with Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor columns)
- Published study reporting manually annotated compound lists and significance patterns (e.g. ANOSIM p-values) for the same samples

## Outputs

- Aggregated compound-by-sample component-area table from mzExacto
- Standardized-area table (standardifyIt) relative to internal or external standards
- Reproduced compound identity list comparable to the manual peak annotations
- Reproduced NMDS/ANOSIM significance pattern for the same treatment groups

## How to apply

Take a previously published GC-MS dataset (e.g. the Ponce et al. 2022 grain dataset of 35 samples across four treatment groups) and re-run the automated pipeline on its hit table. In uafR, load the published hit table with read.csv, run spreadOut() on the six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), then run mzExacto(spread, query_compounds, decontaminate = TRUE) to aggregate candidate peak areas, dropping chemicals without a strong cross-sample match, no PubChem entry, or no canonical SMILES. Derive the query list from categorate output via exactoThese(subsetBy = 'Database', subsetArgs = c('LOTUS', 'KEGG')) and enrich with categorate(detail = 'research') to support all four identification methods. Standardize component areas against known internal or external standards with standardifyIt(). The concordance test is two-fold: first compare the number and types of compounds identified against the published manual peak annotations, and second reproduce the downstream ordination/permutation statistics (NMDS, ANOSIM) and check that the pattern of significance matches the original study.

## Related tools

- **uafR** (R package whose GC-MS workflow (spreadOut, mzExacto, standardifyIt) re-processes the published hit table so its output can be compared to the manual peak annotations) — https://github.com/castratton/uafR
- **Agilent Unknowns Analysis** (Vendor deconvolution software that produced the tentative peak identifications which both the original manual annotation and the uafR re-analysis build upon)
- **PubChem** (Open-source chemical database used by mzExacto decontamination to drop chemicals lacking a PubChem entry or canonical SMILES, keeping the re-analysis concordant with published compound types)

## Examples

```
input_dat = read.csv("your/gcms/dataset.csv"); input_spread = spreadOut(input_dat); input_exacto = mzExacto(input_spread, query_chemicals, decontaminate = TRUE)
```

## Evaluation signals

- The number and types of compounds identified by uafR are comparable (or identical) to those identified by the traditional manual peak-annotation process reported in the original study
- Retention-time matches for the re-identified compounds are accurate relative to the published annotations
- ANOSIM (or equivalent) analysis of the uafR-identified compounds reproduces the same overall pattern of significance as the original study
- The input hit table passed spreadOut validation: all six required columns present, no duplicate peak keys, no missing values in the retained columns
- Match factors and retention times are within sensible ranges and area distributions inspected for anomalies before comparison

## Limitations

- uafR has not yet been tested in contexts other than the GC-MS/QQQ data for which it was developed (e.g. Q-TOF, Orbitrap, TimsTOF HT, Astral, LC-MS)
- The algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data because the spectra are not yet published for most chemicals in PubChem's database
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [abstract] Re-running uafR on the published Ponce et al. 2022 grain dataset reproduced manual identifications with accurate retention-time matches and yielded the same overall pattern of significance in ANOSIM analysis.: "the number and types of compounds identified were comparable (or identical) to those identified with the traditional manual peak annotation process, and NMDS analysis of the compounds produced the"
- [intro] Objective: validate the package's accuracy and efficiency against a known-standards dataset and a previously published dataset: "Here, we present two examples to demonstrate the accuracy and efficiency of uafR"
- [methods] The re-analysis workflow uses the vendor hit table, required columns, and decontamination filter as described in the methods.: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [abstract] The downstream statistics reproduced the same significance pattern as the original study, confirming concordance with manual annotation.: "NMDS analysis of the compounds produced the same pattern of significance as in the original study"
- [intro] The comparison target is the published manual peak-annotation process, which defines what 'concordant' means.: "The traditional methods involve manually selecting, integrating, and identifying peaks based on a reference library and comparison to commercial standards"
