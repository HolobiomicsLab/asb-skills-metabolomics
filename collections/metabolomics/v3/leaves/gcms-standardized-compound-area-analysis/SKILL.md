---
name: gcms-standardized-compound-area-analysis
description: Use when uafR has produced an aggregated GC-MS compound-area table from
  Agilent Unknowns Analysis output and the goal is a between-treatment NMDS significance
  analysis that must match a published result.
when_to_use_negative:
- The goal is raw peak identification or extraction rather than downstream statistics
  — use spreadOut()/mzExacto() directly instead
- Data are MS2, MS3, or ion mobility + MS2 — uafR's algorithms depend on published
  spectra and do not apply to these acquisition modes
- The comparison target is compound identity accuracy against known standards rather
  than between-treatment community/composition significance
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: R package whose downstream statistical analysis pathway standardizes compound
    areas and runs NMDS ordination on the GC-MS compound-area matrix
  repo: https://github.com/castratton/uafR
- name: R
  role: Environment in which the dissimilarity computation, NMDS ordination, and between-treatment
    significance test are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:7fe4dd3a075ebe5eb22c8ada6d74cf06b5b48aa1a303c4809ab5ffb50baf789e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - R
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gcms-standardized-compound-area-analysis

## Summary

Standardizes GC-MS component areas from uafR's peak-extraction output relative to internal or external standards, computes sample-by-sample dissimilarities, and runs NMDS ordination with a between-treatment significance test to reproduce published between-treatment statistical patterns.

## When to use

Use when uafR has produced an aggregated GC-MS compound-area table from Agilent Unknowns Analysis output and the goal is a between-treatment NMDS significance analysis that must match a published result.

## When NOT to use

- The goal is raw peak identification or extraction rather than downstream statistics — use spreadOut()/mzExacto() directly instead
- Data are MS2, MS3, or ion mobility + MS2 — uafR's algorithms depend on published spectra and do not apply to these acquisition modes
- The comparison target is compound identity accuracy against known standards rather than between-treatment community/composition significance

## Inputs

- Standardized compound-area table from the uafR re-analysis of the Ponce et al. 2022 GC-MS samples (columns: Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor)
- Treatment/group assignment per sample (File.Name key)
- Internal or external standard values for area standardization

## Outputs

- NMDS ordination scores for samples
- Between-treatment significance test statistics and p-values
- nmds_significance_results.csv
- packageVersion and sessionInfo audit record

## How to apply

Load the aggregated compound-area table produced by uafR's spreadOut()/mzExacto() pipeline (requiring columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and standardize component areas relative to known internal or external standards. Compute sample-by-sample dissimilarities from the standardized compound-area matrix in R using uafR's downstream statistical pathway. Run non-metric multidimensional scaling (NMDS) on the compound-area matrix, mirroring the original Ponce et al. 2022 analysis pipeline. Test between-treatment differences in compound composition using the same statistical framework as the original study, then write ordination scores plus test statistics and p-values to nmds_significance_results.csv. The validation criterion is that the reproduced NMDS yields the same pattern of significance between treatments as reported in the original study; record packageVersion and sessionInfo alongside the result for audit.

## Related tools

- **uafR** (R package whose downstream statistical analysis pathway standardizes compound areas and runs NMDS ordination on the GC-MS compound-area matrix) — https://github.com/castratton/uafR
- **R** (Environment in which the dissimilarity computation, NMDS ordination, and between-treatment significance test are executed)

## Examples

```
spread <- spreadOut(read.csv('ponce2022_gcms.csv')); extracted <- mzExacto(spread, targets, decontaminate = TRUE); write.csv(nmds_results, 'nmds_significance_results.csv')
```

## Evaluation signals

- Reproduced NMDS analysis produces the same pattern of significance between treatments as reported in the original Ponce et al. 2022 study
- Compound areas are standardized relative to known internal or external standards before ordination
- nmds_significance_results.csv contains both ordination scores and between-treatment test statistics with p-values
- packageVersion and sessionInfo are recorded alongside the result for audit
- Number and types of compounds entering the analysis are comparable (or identical) to the traditional manual peak-annotation process

## Limitations

- Manual validation is still required: researchers should confirm that compound areas identified by the algorithm make chemical and/or biological sense in their study system
- Reproducibility depends on recording the instrument, library, search settings, filters, and software versions
- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS data

## Evidence

- [abstract] NMDS reproduces published significance pattern: "NMDS analysis of the compounds produced the same pattern of significance as in the original study"
- [intro] Standardization against internal/external standards: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [intro] Downstream statistics on compound areas: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [readme] standardifyIt standardizes abundance in GC-MS workflow: [evidence span withheld — anchor status: external_doc]
- [readme] Input column requirements for the peak table: [evidence span withheld — anchor status: external_doc]
