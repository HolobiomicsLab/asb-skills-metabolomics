---
name: cross-study-significance-pattern-verification
description: Use when an automated GC-MS re-analysis of an already-published dataset
  must be validated by reproducing the original study's between-treatment significance
  pattern (e.g., NMDS of compound areas).
when_to_use_negative:
- The source data are MS2, MS3, or ion mobility + MS2 acquisitions — uafR's algorithms
  depend on published spectra and do not apply to these contexts
- Data were acquired on instruments uafR has not been tested on (Q-TOF, Orbitrap,
  TimsTOF HT, Astral) or are LC-MS rather than GC-MS data
- There is no previously published analysis with a reported significance pattern to
  compare against, or the original study used a different compound-annotation framework
  whose design cannot be mirrored
edam_operation: http://edamontology.org/operation_0306
edam_topics: []
tools:
- name: uafR
  role: Automates GC-MS component-area retrieval, decontamination of tentative identifications,
    and standardization of areas relative to internal/external standards before ordination
    and significance testing
  repo: github.com/castratton/uafR
- name: R
  role: Environment in which the dissimilarity computation, NMDS ordination, and between-treatment
    significance test are run via uafR's downstream statistical analysis pathway
- name: devtools
  role: Installs the uafR package directly from R or RStudio via devtools::install_github("castratton/uafR")
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:82f426cbe377c9d49cbc45aa06e87185c69983a8485cae6992263a2081840b4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-study-significance-pattern-verification

## Summary

Re-runs an ordination-based between-treatment test (NMDS on GC-MS compound areas processed by uafR) against a previously published study's statistical framework to verify that the automated pipeline reproduces the original pattern of significance.

## When to use

Use when an automated GC-MS re-analysis of an already-published dataset must be validated by reproducing the original study's between-treatment significance pattern (e.g., NMDS of compound areas).

## When NOT to use

- The source data are MS2, MS3, or ion mobility + MS2 acquisitions — uafR's algorithms depend on published spectra and do not apply to these contexts
- Data were acquired on instruments uafR has not been tested on (Q-TOF, Orbitrap, TimsTOF HT, Astral) or are LC-MS rather than GC-MS data
- There is no previously published analysis with a reported significance pattern to compare against, or the original study used a different compound-annotation framework whose design cannot be mirrored

## Inputs

- Raw aggregated GC-MS peak table CSV from vendor peak-detection output (required columns: Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor)
- Standardized compound-area table from the uafR re-analysis (areas standardized relative to internal or external standards)
- Sample treatment/replicate metadata matching the original published study's design

## Outputs

- nmds_significance_results.csv containing NMDS ordination scores, between-treatment test statistics, and p-values
- packageVersion and sessionInfo records for provenance/audit

## How to apply

Load the standardized compound-area table produced by the uafR re-analysis of the published GC-MS samples, having standardized component areas relative to known internal or external standard(s). First prepare the raw vendor export (e.g., Agilent Unknowns Analysis output with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) via spreadOut(), then extract targets with mzExacto(..., decontaminate = TRUE) to remove chemicals lacking strong cross-sample matches, a PubChem entry, or a canonical SMILES. Compute sample-by-sample dissimilarities from the standardized compound-area matrix in R and run a non-metric multidimensional scaling (NMDS) ordination exactly as in the original study's pipeline. Test between-treatment differences using the same statistical framework as the original paper, then write the NMDS ordination scores plus test statistics and p-values to nmds_significance_results.csv. Validation is by comparison: the reproduced analysis must yield the same pattern of significance between treatments as reported in the original study, with packageVersion and sessionInfo recorded alongside for audit.

## Related tools

- **uafR** (Automates GC-MS component-area retrieval, decontamination of tentative identifications, and standardization of areas relative to internal/external standards before ordination and significance testing) — github.com/castratton/uafR
- **R** (Environment in which the dissimilarity computation, NMDS ordination, and between-treatment significance test are run via uafR's downstream statistical analysis pathway)
- **devtools** (Installs the uafR package directly from R or RStudio via devtools::install_github("castratton/uafR"))

## Examples

```
selected = mzExacto(spread, query_compounds, decontaminate = TRUE)
```

## Evaluation signals

- The reproduced NMDS analysis produces the same pattern of significance between treatments as reported in the original published study
- The number and types of compounds identified are comparable (or identical) to those from the traditional manual peak-annotation process
- Decontamination was applied (chemicals lacking strong cross-sample match, PubChem entry, or canonical SMILES removed) and a minimal Match.Factor threshold was set on the raw Unknowns Analysis output
- packageVersion and sessionInfo are recorded alongside nmds_significance_results.csv for audit
- Upstream quantification quality was verified (e.g., peak areas for internal standards correlated with injected volume, R2 range 0.8273–0.9998)

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or on LC-MS data
- The algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data; functionality will be added once these spectra are published for most chemicals in PubChem's database

## Evidence

- [abstract] Reproduce published significance pattern: "the number and types of compounds identified were comparable (or identical) to those identified with the traditional manual peak annotation process, and NMDS analysis of the compounds produced the"
- [intro] Downstream statistics in R: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [intro] Area standardization: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [methods] Decontamination call: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [readme] Required input schema: [evidence span withheld — anchor status: external_doc]
