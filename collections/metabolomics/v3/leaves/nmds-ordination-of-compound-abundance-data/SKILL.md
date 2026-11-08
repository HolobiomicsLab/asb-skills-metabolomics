---
name: nmds-ordination-of-compound-abundance-data
description: Use when a standardized sample-by-compound area table from uafR GC-MS
  processing is available and the goal is ordination plus a between-treatment significance
  test of compound composition, e.g. reproducing a published result.
when_to_use_negative:
- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms do not
  yet apply since these spectra are not widely published in PubChem's database
- You need automated peak annotation or deconvolution from raw vendor output — this
  skill starts from an already-extracted, standardized compound-area table produced
  via spreadOut()/mzExacto()
- The goal is unstandardized relative peak-area comparison — areas must first be standardized
  against known internal or external standards
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Supplies the standardized compound-area matrix (via spreadOut(), mzExacto(),
    standardifyIt()) and the downstream statistical analysis pathway used to compute
    dissimilarities and run NMDS
  repo: https://github.com/castratton/uafR
- name: R
  role: Analysis environment in which the NMDS ordination and between-treatment significance
    testing are executed
- name: RStudio
  role: Environment from which uafR is installed with devtools before running the
    ordination workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:24bdeaf666e10e9ab0381289bc8e30c202fbce4d5a7bc6b343914c168da78ad3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - R
  - RStudio
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nmds-ordination-of-compound-abundance-data

## Summary

Runs non-metric multidimensional scaling (NMDS) ordination on uafR-derived, standardized GC-MS compound-area matrices and tests between-treatment differences, the downstream statistical step that reproduced the Ponce et al. 2022 significance pattern.

## When to use

Use when a standardized sample-by-compound area table from uafR GC-MS processing is available and the goal is ordination plus a between-treatment significance test of compound composition, e.g. reproducing a published result.

## When NOT to use

- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms do not yet apply since these spectra are not widely published in PubChem's database
- You need automated peak annotation or deconvolution from raw vendor output — this skill starts from an already-extracted, standardized compound-area table produced via spreadOut()/mzExacto()
- The goal is unstandardized relative peak-area comparison — areas must first be standardized against known internal or external standards

## Inputs

- Standardized compound-area table from uafR re-analysis (CSV whose columns must include Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), areas standardized relative to internal/external standards
- Sample/treatment metadata mapping File.Name to experimental treatments (e.g. the Ponce et al. 2022 GC-MS sample set)

## Outputs

- NMDS ordination scores for samples
- Between-treatment significance test statistics and p-values
- nmds_significance_results.csv containing scores, statistics, and p-values
- packageVersion and sessionInfo audit record

## How to apply

Load the standardized compound-area table produced by uafR (areas standardized relative to known internal or external standards) into R. Compute sample-by-sample dissimilarities from the standardized compound areas using uafR's downstream statistical analysis pathway, then run NMDS ordination on the compound-area matrix as in the original study's pipeline. Test between-treatment differences in compound composition using the same statistical framework as the original study, and write the ordination scores, test statistics, and p-values to nmds_significance_results.csv. Record packageVersion and sessionInfo alongside the result for audit, since the validation criterion is that the reproduced NMDS matches the original study's pattern of significance.

## Related tools

- **uafR** (Supplies the standardized compound-area matrix (via spreadOut(), mzExacto(), standardifyIt()) and the downstream statistical analysis pathway used to compute dissimilarities and run NMDS) — https://github.com/castratton/uafR
- **R** (Analysis environment in which the NMDS ordination and between-treatment significance testing are executed)
- **RStudio** (Environment from which uafR is installed with devtools before running the ordination workflow)

## Evaluation signals

- The reproduced NMDS analysis produces the same pattern of significance between treatments as reported in the original Ponce et al. 2022 study
- nmds_significance_results.csv contains ordination scores plus between-treatment test statistics and p-values
- packageVersion and sessionInfo are recorded alongside the result for audit
- The compound areas fed into the NMDS were validated to make chemical and/or biological sense in the study system
- The compound count and identities entering the ordination are comparable (or identical) to those from traditional manual peak annotation

## Limitations

- uafR has not yet been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data, so NMDS of such data is out of scope
- Algorithms do not apply to MS2, MS3, or ion mobility + MS2 data; functionality will be added once these spectra are published for most chemicals in PubChem's database
- Researchers must still manually validate that compound areas identified by the algorithm make chemical and/or biological sense before trusting downstream NMDS results

## Evidence

- [abstract] NMDS analysis of the compounds produced the same pattern of significance as in the original study: "NMDS analysis of the compounds produced the same pattern of significance as in the original study"
- [intro] including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [readme] Prepare raw peak tables with `spreadOut()`, extract target compounds with `mzExacto()`, and standardize abundance with `standardifyIt()`.: [evidence span withheld — anchor status: external_doc]
- [discussion] researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
