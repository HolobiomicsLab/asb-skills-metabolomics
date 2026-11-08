---
name: multivariate-ordination-input-preparation
description: Use when you have an aggregated peak-area table from GC-MS library hits
  (e.g. mzExacto output) and need standardized, statistically comparable component
  areas for downstream ordination or modeling.
when_to_use_negative:
- Input is already a standardized or normalized feature table ready for ordination
- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published
  fragmentation data and do not yet apply
- The goal is chemical annotation or compound selection (use categorate()/exactoThese())
  rather than abundance standardization
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Provides spreadOut(), mzExacto(), and standardifyIt() — the upstream spread/aggregate
    functions and the standardization routine that formats peak areas for ordination
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment in which the uafR standardization and downstream ordination
    workflow is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:86d3ae77eb6abfc5076e86135c254e9cb0e763b71839358c1c73bc963d0b1a06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# multivariate-ordination-input-preparation

## Summary

Standardizes GC-MS component areas of tentatively identified chemicals relative to known internal or external standards and formats the result as an ordination-ready table for PCA, NMDS, or machine-learning analyses in the uafR workflow.

## When to use

Use when you have an aggregated peak-area table from GC-MS library hits (e.g. mzExacto output) and need standardized, statistically comparable component areas for downstream ordination or modeling.

## When NOT to use

- Input is already a standardized or normalized feature table ready for ordination
- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published fragmentation data and do not yet apply
- The goal is chemical annotation or compound selection (use categorate()/exactoThese()) rather than abundance standardization

## Inputs

- mzExacto() output data frame of aggregated component areas for tentatively identified chemicals across GC-MS samples
- User-specified internal standard identity (e.g. tetradecane) or external-standard calibration curve values quantifying known chemical amounts
- Tuning sub-argument values describing the experimental methods

## Outputs

- Data frame of component areas standardized relative to known internal or external standard quantifications
- Ordination-ready table formatted for PCA, NMDS, and/or machine-learning algorithms

## How to apply

Start from the mzExacto() output, which aggregates candidate peak areas across samples for the query compounds (e.g. spread from spreadOut() with decontaminate = TRUE). Supply either a user-specified internal standard (e.g. tetradecane) or external-standard calibration curves quantifying known chemical amounts, then run standardifyIt() with its tuning sub-arguments set to match the experimental method. standardifyIt() normalizes every component area relative to the known standard quantifications and returns a data frame formatted for subsequent statistical analyses. Inspect the standardized table (row counts, duplicate keys, match factors, area distributions) before feeding it to PCA, NMDS, or machine-learning algorithms, and validate that standardized areas respond linearly to injected standard quantity.

## Related tools

- **uafR** (Provides spreadOut(), mzExacto(), and standardifyIt() — the upstream spread/aggregate functions and the standardization routine that formats peak areas for ordination) — https://github.com/castratton/uafR
- **R** (Runtime environment in which the uafR standardization and downstream ordination workflow is executed)

## Examples

```
standardized <- standardifyIt(mzExacto(spreadOut(raw_gcms), targets, decontaminate = TRUE), internal_standard = "tetradecane")
```

## Evaluation signals

- Standardized peak areas show linear response to injected standard quantity, consistent with the package's reported R2 range of 0.8273–0.9998 across 1e-5 to 1e3 ng/µl
- Standardized data frame retains the expected input row counts with no unintended duplicate keys and match factors consistent with the mzExacto output
- Area distributions after standardization are inspected and confirmed to make chemical and/or biological sense in the study system
- Downstream NMDS reproduces the same significance pattern as a manual peak-annotation baseline, as demonstrated in the paper's published-dataset validation

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data
- Algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data since they depend on spectra published in PubChem

## Evidence

- [intro] Standardize component areas relative to internal or external standards: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [intro] Downstream analyses: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [results] Standard-area linearity validation: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998"
- [discussion] Manual validation still required: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
- [readme] standardifyIt in GC-MS workflow: [evidence span withheld — anchor status: external_doc]
