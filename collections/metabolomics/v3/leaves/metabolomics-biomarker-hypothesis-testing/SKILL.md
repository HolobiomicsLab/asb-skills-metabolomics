---
name: metabolomics-biomarker-hypothesis-testing
description: Use when a two-group metabolomics comparison requires simultaneous multiple-hypothesis
  testing across all metabolite features with calibrated fdr2d cutoff rejection counts.
when_to_use_negative:
- The raw (unprocessed) spectra have not yet passed MetPC's metabolite-identification/preprocessing
  steps — the fdr2d test expects the processed feature matrix
- The goal is metabolite identification via the hierarchical statistical model with
  EM algorithm rather than biomarker hypothesis testing
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetPC
  role: Implements the fdr2d-controlled biomarker discovery procedure used to test
    all 3226 metabolite null hypotheses and compute rejection counts at fdr2d levels
    0.05-0.3
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:3ab796a176e9fafd085fc168b4b8477866c7a910bfc401c9b8eaffc292a0062a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-biomarker-hypothesis-testing

## Summary

Biomarker discovery step of the MetPC pipeline: tests null hypotheses for all metabolite features simultaneously (e.g. Korea vs China comparison of a 3226 x 57 schisandra metabolite matrix) while controlling the two-dimensional false discovery rate (fdr2d).

## When to use

Use when a two-group metabolomics comparison requires simultaneous multiple-hypothesis testing across all metabolite features with calibrated fdr2d cutoff rejection counts.

## When NOT to use

- The raw (unprocessed) spectra have not yet passed MetPC's metabolite-identification/preprocessing steps — the fdr2d test expects the processed feature matrix
- The goal is metabolite identification via the hierarchical statistical model with EM algorithm rather than biomarker hypothesis testing

## Inputs

- Processed metabolomics data matrix of 3226 metabolite features x 57 samples (schisandra, Korea vs China groups)

## Outputs

- Rejected-hypothesis counts per fdr2d cutoff level (0.05, 0.1, 0.2, 0.3), written to fdr2d_rejections.csv

## How to apply

Load the preprocessed metabolomics data matrix (here 3226 metabolite features x 57 samples) and run the two-dimensional local FDR (fdr2d) biomarker-discovery procedure from MetPC over the null hypotheses of all 3226 features simultaneously. Apply the fdr2d thresholding rule at the four cutoff levels 0.05, 0.1, 0.2, and 0.3 to determine how many null hypotheses are rejected at each level. Write the per-cutoff rejected-hypothesis counts to fdr2d_rejections.csv. The rationale, per the MetPC paper, is that the procedure controls the two-dimensional false discovery rate (fdr2d) during simultaneous multiple-hypothesis testing for biomarker metabolite discovery.

## Related tools

- **MetPC** (Implements the fdr2d-controlled biomarker discovery procedure used to test all 3226 metabolite null hypotheses and compute rejection counts at fdr2d levels 0.05-0.3) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript fdr2d_rejections.R --input schisandra_3226x57.csv --levels 0.05,0.1,0.2,0.3 --out fdr2d_rejections.csv
```

## Evaluation signals

- Rejection count at fdr2d level 0.05 must equal 1092
- Rejection counts at fdr2d levels 0.1, 0.2, 0.3 must equal 1326, 1782, and 2120 respectively
- The hypothesis set must cover all 3226 metabolite features simultaneously (no subsetting)
- Output file fdr2d_rejections.csv must contain one count per requested cutoff level (0.05, 0.1, 0.2, 0.3)
- Rejection counts must be monotonically non-decreasing as the fdr2d level increases

## Limitations

- Validated only on the 3226 x 57 schisandra Korea-vs-China comparison described in the task; counts for other datasets will differ
- The article text provides no discussion-section detail; guidance beyond the abstract-level description of the fdr2d control is unavailable from the indexed content

## Evidence

- [abstract] Biomarker metabolite discovery controls the two-dimensional false discovery rate (fdr2d) in simultaneous multiple hypothesis testing.: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously"
- [abstract] The fdr2d step is one of two pillars of the MetPC pipeline alongside metabolite identification.: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
