---
name: multiple-comparison-significance-filtering
description: Use when you have a binary presence/absence detection matrix across cohort
  samples and need to identify statistically significant co-detection pairs (e.g.,
  parent compounds vs. their CMDN-annotated metabolites) among many tested pairs.
when_to_use_negative:
- Input data are continuous abundance/intensity values rather than binary detection
  calls — the 2x2 contingency-table construction requires qualitative presence/absence
  coding
- The goal is quantitative correlation of exposure levels rather than qualitative
  co-occurrence of detection events
- Testing a single pre-specified pair only, where multiple-comparison filtering across
  a large pair space is not the concern
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: CMDN
  role: Provides the CMDN-annotated metabolite labels used to define the metabolite
    side of each parent–metabolite pair tested for co-occurrence
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:45810965705f331f1a0bb3fb6bcc67305fcd40f369b107784615c8c08052f956
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# multiple-comparison-significance-filtering

## Summary

Applies Chi-squared tests of independence to 2x2 contingency tables built from binary detection/non-detection data, flagging parent pesticide–metabolite co-occurrence pairs at p < 0.05 to control which exposure linkages are reported from a large pairwise testing space.

## When to use

Use when you have a binary presence/absence detection matrix across cohort samples and need to identify statistically significant co-detection pairs (e.g., parent compounds vs. their CMDN-annotated metabolites) among many tested pairs.

## When NOT to use

- Input data are continuous abundance/intensity values rather than binary detection calls — the 2x2 contingency-table construction requires qualitative presence/absence coding
- The goal is quantitative correlation of exposure levels rather than qualitative co-occurrence of detection events
- Testing a single pre-specified pair only, where multiple-comparison filtering across a large pair space is not the concern

## Inputs

- Binary presence/absence detection matrix (samples x compounds) for parent pesticide compounds and CMDN-annotated metabolites from human cohort urine LC-HRMS screening results
- List of parent compound–metabolite pairings (e.g., carbaryl vs. +H2O+H2O derivative; fenoxycarb vs. +O derivative)

## Outputs

- Per-pair 2x2 contingency tables of joint detection/non-detection counts
- Chi-squared statistics and p-values for every parent–metabolite pair
- Summary table of significant positive co-occurrence associations (chi_squared_correlation_results.csv)

## How to apply

Load the binary presence/absence detection matrix for parent compounds and CMDN-annotated metabolites from the human cohort urine screening results with pandas. For every parent compound–metabolite pair, construct a 2x2 contingency table of joint detection/non-detection counts across cohort samples, then apply scipy.stats.chi2_contingency to test qualitative co-occurrence. Flag associations as significantly positive at p < 0.05 and compile the full co-occurrence landscape with chi-squared statistics and p-values (e.g., to chi_squared_correlation_results.csv). Because every parent–metabolite pair is tested, the p < 0.05 threshold acts as the multiple-comparison significance filter that decides which linkages survive; validate by confirming the reproduced pairs recover the article's reported associations (carbaryl: +H2O+H2O, p = 0.018; fenoxycarb: +O, p = 0.03).

## Related tools

- **CMDN** (Provides the CMDN-annotated metabolite labels used to define the metabolite side of each parent–metabolite pair tested for co-occurrence) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
from scipy.stats import chi2_contingency; stat, p, dof, _ = chi2_contingency(table_2x2); sig = pairs[p < 0.05]
```

## Evaluation signals

- Reproduced analysis recovers carbaryl: +H2O+H2O at p = 0.018 and fenoxycarb: +O at p = 0.03, matching the article's reported associations
- Output table chi_squared_correlation_results.csv contains chi-squared statistics and p-values for every parent–metabolite pair tested, not only significant ones
- Every flagged association satisfies p < 0.05
- Contingency tables are valid 2x2 joint detection/non-detection counts summing to the cohort sample size for each pair

## Limitations

- Chi-squared testing of many parent–metabolite pairs inflates false positives; the p < 0.05 threshold is applied per-pair without explicit correction described in the workflow
- Qualitative co-occurrence does not establish causality or dose–response between parent compound and metabolite — it is a screening-level exposure-linkage signal ('suggesting a high coexposure risk')

## Evidence

- [results] Chi-squared co-occurrence analysis of parent–metabolite pairs at p < 0.05: [evidence span withheld — anchor status: not_found]
