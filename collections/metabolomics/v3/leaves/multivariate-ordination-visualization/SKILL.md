---
name: multivariate-ordination-visualization
description: Use when a standardized GC-MS compound-area matrix (areas relative to
  internal/external standards) must be ordinated with NMDS and the between-treatment
  significance pattern checked against a published manual-peak-annotation analysis.
when_to_use_negative:
- The input data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms do
  not apply to these data types
- The data are from untested instrument/data contexts such as LC-MS, Q-TOF, Orbitrap,
  TimsTOF HT, or Astral
- The goal is de novo peak deconvolution or library matching — use Agilent Unknowns
  Analysis (deconvolution algorithm) upstream of this skill instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: uafR
  role: Provides the downstream statistical analysis pathway used to compute sample-by-sample
    dissimilarities and run NMDS on the standardized compound-area matrix
  repo: github.com/castratton/uafR
- name: R
  role: Execution environment for running the NMDS ordination, between-treatment significance
    tests, and writing nmds_significance_results.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:b0016e6b6bdb66aa58b61d05ffafe45bb83891e7095882cfb5870be5fbfe83e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multivariate-ordination-visualization

## Summary

Runs non-metric multidimensional scaling (NMDS) ordination on standardized GC-MS compound-area tables via uafR's downstream statistical pathway, then tests between-treatment differences and audits the result against the originally published significance pattern (Ponce et al. 2022).

## When to use

Use when a standardized GC-MS compound-area matrix (areas relative to internal/external standards) must be ordinated with NMDS and the between-treatment significance pattern checked against a published manual-peak-annotation analysis.

## When NOT to use

- The input data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms do not apply to these data types
- The data are from untested instrument/data contexts such as LC-MS, Q-TOF, Orbitrap, TimsTOF HT, or Astral
- The goal is de novo peak deconvolution or library matching — use Agilent Unknowns Analysis (deconvolution algorithm) upstream of this skill instead

## Inputs

- Standardized compound-area table from uafR re-analysis of GC-MS samples (areas standardized relative to internal or external standards)
- Treatment/group labels per sample matching the original study design
- Published between-treatment significance results to reproduce (Ponce et al. 2022)

## Outputs

- nmds_significance_results.csv containing NMDS ordination scores and between-treatment significance test statistics and p-values
- packageVersion and sessionInfo records for audit

## How to apply

Start from the standardized compound-area table produced by uafR's GC-MS workflow (spreadOut → mzExacto → standardifyIt), where areas are quantified relative to known internal or external standard(s). Compute sample-by-sample dissimilarities from the standardized compound areas in R using uafR's downstream statistical analysis pathway, which also supports principal component analyses and machine learning algorithms. Run NMDS ordination on the compound-area matrix using the same statistical framework as the original study, then test between-treatment differences in compound composition. Write the NMDS ordination scores plus between-treatment significance test statistics and p-values to nmds_significance_results.csv, recording packageVersion and sessionInfo alongside the result for audit. Validation is a direct reproduction check: the NMDS analysis must produce the same pattern of significance between treatments as reported in the original study.

## Related tools

- **uafR** (Provides the downstream statistical analysis pathway used to compute sample-by-sample dissimilarities and run NMDS on the standardized compound-area matrix) — github.com/castratton/uafR
- **R** (Execution environment for running the NMDS ordination, between-treatment significance tests, and writing nmds_significance_results.csv)

## Examples

```
library(uafR); spread <- spreadOut(input_dat); extracted <- mzExacto(spread, query_chemicals, decontaminate = TRUE); standardized <- standardifyIt(extracted)
```

## Evaluation signals

- Reproduced NMDS analysis produces the same pattern of significance between treatments as reported in the original Ponce et al. 2022 study
- nmds_significance_results.csv contains NMDS ordination scores plus between-treatment test statistics and p-values
- packageVersion and sessionInfo are recorded alongside the result for audit
- Number and types of compounds feeding the ordination are comparable (or identical) to those from the traditional manual peak annotation process
- Component areas were standardized relative to known internal or external standard(s) before dissimilarity computation

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS data
- Algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data

## Evidence

- [abstract] NMDS analysis of the compounds produced the same pattern of significance as in the original study: "NMDS analysis of the compounds produced the same pattern of significance as in the original study"
- [intro] Standardization of areas relative to standards precedes statistics: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [intro] uafR supports PCA, NMDS, and machine learning downstream analyses: "including principal component analyses, non-metric multidimensional scaling (NMDS), and/or machine learning algorithms"
- [readme] README describes the GC-MS preprocessing functions feeding the ordination: [evidence span withheld — anchor status: external_doc]
- [discussion] Manual validation of compound areas still required: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
