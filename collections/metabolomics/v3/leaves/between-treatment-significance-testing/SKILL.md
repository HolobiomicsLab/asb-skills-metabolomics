---
name: between-treatment-significance-testing
description: Use when a standardized compound-area table from GC-MS peak extraction
  exists and the goal is to test whether compound composition differs between treatments,
  e.g. reproducing a published study's NMDS-based significance pattern.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms do not yet apply
  to those data types
- No standardized compound-area table exists yet (areas not standardized relative
  to internal/external standards), so the ordination input would be raw, incomparable
  peak areas
- The dataset comes from instruments uafR has not been tested on (e.g. Q-TOF, Orbitrap,
  TimsTOF HT, Astral or LC-MS data)
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: uafR
  role: Provides the standardized compound-area table via spreadOut(), mzExacto(decontaminate
    = TRUE), and standardifyIt(), then feeds its downstream statistical analysis pathway
    for NMDS ordination and significance testing
  repo: github.com/castratton/uafR
- name: R
  role: Runtime for computing sample-by-sample dissimilarities, running the NMDS ordination,
    and executing the between-treatment significance tests on the compound-area matrix
- name: devtools
  role: Installs uafR directly from R or RStudio via devtools::install_github('castratton/uafR')
    before the statistical workflow is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:95a06271baf769d12185efad4ae9de53e3fa99f967b061d2687626de2bf0ac5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# between-treatment-significance-testing

## Summary

Reproduces between-treatment statistical significance testing of GC-MS compound composition by running NMDS ordination on standardized compound-area tables through uafR's downstream statistical pathway in R, validating against a published study's reported significance pattern.

## When to use

Use when a standardized compound-area table from GC-MS peak extraction exists and the goal is to test whether compound composition differs between treatments, e.g. reproducing a published study's NMDS-based significance pattern.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms do not yet apply to those data types
- No standardized compound-area table exists yet (areas not standardized relative to internal/external standards), so the ordination input would be raw, incomparable peak areas
- The dataset comes from instruments uafR has not been tested on (e.g. Q-TOF, Orbitrap, TimsTOF HT, Astral or LC-MS data)

## Inputs

- Standardized compound-area table from uafR GC-MS processing (areas standardized relative to internal or external standards)
- Raw vendor GC-MS export CSV with required columns: Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Treatment/group metadata per sample file for between-treatment comparison

## Outputs

- nmds_significance_results.csv containing NMDS ordination scores and between-treatment significance test statistics and p-values
- packageVersion and sessionInfo records for reproducibility audit

## How to apply

Load the standardized compound-area table produced by uafR's extraction pathway (spreadOut -> mzExacto with decontaminate = TRUE -> standardifyIt), where component areas were quantified relative to known internal or external standards. Compute sample-by-sample dissimilarities from the standardized compound areas and run a non-metric multidimensional scaling (NMDS) ordination on the compound-area matrix, using the same statistical framework as the original study being reproduced. Test between-treatment differences in compound composition with the same test statistics the original study used. Write the NMDS ordination scores and between-treatment significance test statistics and p-values to nmds_significance_results.csv. Validation requires that the reproduced NMDS analysis produce the same pattern of significance between treatments as reported in the original study; record packageVersion and sessionInfo alongside the result for audit.

## Related tools

- **uafR** (Provides the standardized compound-area table via spreadOut(), mzExacto(decontaminate = TRUE), and standardifyIt(), then feeds its downstream statistical analysis pathway for NMDS ordination and significance testing) — github.com/castratton/uafR
- **R** (Runtime for computing sample-by-sample dissimilarities, running the NMDS ordination, and executing the between-treatment significance tests on the compound-area matrix)
- **devtools** (Installs uafR directly from R or RStudio via devtools::install_github('castratton/uafR') before the statistical workflow is run)

## Examples

```
spread <- spreadOut(raw_gcms); extracted <- mzExacto(data_in = spread, chemicals = targets, decontaminate = TRUE); standardized <- standardifyIt(extracted)
```

## Evaluation signals

- The reproduced NMDS analysis produces the same pattern of significance between treatments as reported in the original study (the Ponce et al. 2022 validation criterion)
- nmds_significance_results.csv exists and contains both NMDS ordination scores and significance test statistics with p-values
- packageVersion and sessionInfo were recorded alongside the results for audit
- Compound areas fed to the ordination were standardized relative to known internal or external standards, not raw integrated areas
- The input peak table passed the column-name check for Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor before extraction

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or on LC-MS data
- The instrument, library, search settings, filters, and software versions must be recorded for a versioned reference; uafR has no changelog available

## Evidence

- [abstract] NMDS reproduced the same significance pattern as the original study validates the whole pathway: "NMDS analysis of the compounds produced the same pattern of significance as in the original study"
- [intro] uafR's downstream statistical analysis scope includes NMDS: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [intro] Areas must be standardized before between-treatment comparison: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [readme] mzExacto output that feeds the ordination matrix: [evidence span withheld — anchor status: external_doc]
