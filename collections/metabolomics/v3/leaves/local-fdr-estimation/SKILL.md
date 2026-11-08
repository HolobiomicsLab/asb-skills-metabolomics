---
name: local-fdr-estimation
description: Use when you must reject null hypotheses for thousands of metabolite
  features at explicit fdr2d cutoffs (e.g. 0.05, 0.1, 0.2, 0.3) and need reproducible
  per-level rejection counts.
when_to_use_negative:
- The data are raw instrument spectra that have not passed the MetPC preprocessing
  steps — the fdr2d step expects the processed feature matrix
- You need metabolite identification rather than biomarker discovery; MetPC handles
  identification separately via a hierarchical statistical model with an EM algorithm
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetPC
  role: Provides the biomarker-discovery step whose fdr2d-controlled simultaneous
    multiple-hypothesis testing is executed to obtain rejection counts
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:cafd041f59ee74e1859844cfe39b29e42c5dc27a7f55459437bf6b0fc513077b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# local-fdr-estimation

## Summary

Applies the two-dimensional local FDR (fdr2d) multiple-testing procedure from the MetPC biomarker-discovery pipeline to control the false discovery rate when simultaneously testing thousands of metabolite hypotheses, e.g. 3226 features in a Korea-vs-China schisandra comparison.

## When to use

Use when you must reject null hypotheses for thousands of metabolite features at explicit fdr2d cutoffs (e.g. 0.05, 0.1, 0.2, 0.3) and need reproducible per-level rejection counts.

## When NOT to use

- The data are raw instrument spectra that have not passed the MetPC preprocessing steps — the fdr2d step expects the processed feature matrix
- You need metabolite identification rather than biomarker discovery; MetPC handles identification separately via a hierarchical statistical model with an EM algorithm

## Inputs

- Processed metabolomics data matrix (features x samples), e.g. 3226 metabolite features x 57 samples
- Group labels defining the two-group comparison (e.g. Korea vs China)
- fdr2d cutoff levels to evaluate (0.05, 0.1, 0.2, 0.3)

## Outputs

- Rejected-hypothesis counts per fdr2d level, written to fdr2d_rejections.csv
- Set of metabolite features (null hypotheses) rejected at each fdr2d level

## How to apply

Load the processed metabolomics data matrix — in the source case, 3226 metabolite features x 57 samples for the Korea-vs-China comparison — and test the null hypothesis for every feature simultaneously using the fdr2d biomarker-discovery step of MetPC. Compute rejections at the four fdr2d levels 0.05, 0.1, 0.2, and 0.3 using the fdr2d thresholding rule, rather than per-feature p-value cutoffs, because the procedure is designed to control the two-dimensional false discovery rate in simultaneous testing. Write the rejected-hypothesis counts per cutoff level to fdr2d_rejections.csv. The expected result pattern for the schisandra data is a monotone increase in rejections with the level: 1092 at 0.05, rising to 1326, 1782, and 2120 at 0.1, 0.2, and 0.3.

## Related tools

- **MetPC** (Provides the biomarker-discovery step whose fdr2d-controlled simultaneous multiple-hypothesis testing is executed to obtain rejection counts) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- Rejection counts at fdr2d level 0.05 must equal 1092 for the 3226-feature schisandra Korea-vs-China matrix
- Rejection counts at levels 0.1, 0.2, and 0.3 must equal 1326, 1782, and 2120 respectively
- Rejection counts must be monotonically non-decreasing as the fdr2d level increases from 0.05 to 0.3
- fdr2d_rejections.csv must contain exactly one count per queried cutoff level (four rows for 0.05, 0.1, 0.2, 0.3)
- Total hypotheses tested must equal the number of rows in the input matrix (3226 for the reference dataset)

## Limitations

- Validated only against the reported schisandra 3226 x 57 Korea-vs-China result; other datasets have no reference rejection counts to verify against
- Requires the preprocessed data matrix from upstream MetPC steps; applying it to unprocessed data is unsupported by the evidence
- The article abstract gives no discussion-section detail, so sensitivity of counts to preprocessing choices is not documented

## Evidence

- [abstract] fdr2d controls the false discovery rate in simultaneous testing: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [abstract] MetPC bundles preprocessing and biomarker discovery: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [other] MetPC repository: "jjs3098/CNU-Bioinformatics-Lab"
