---
name: statistical-reproducibility-verification
description: Use when a published finding reports concrete numerical outputs (e.g.
  rejection counts at fdr2d levels 0.05, 0.1, 0.2, 0.3) that must be re-derived from
  raw processed data to confirm reproducibility.
when_to_use_negative:
- Input is raw, unprocessed spectral data rather than the preprocessed 3226 x 57 feature
  matrix — preprocessing is a separate pipeline step
- The goal is de novo biomarker discovery rather than validating a specific published
  numerical result against reported counts
- The comparison involves more than the two groups (Korea vs China) for which the
  fdr2d rejection counts were reported
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetPC
  role: Provides the biomarker-discovery step implementing the two-dimensional local
    FDR (fdr2d) procedure used to compute rejections across the four cutoff levels
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:d0a0707f48e849643c075587ede39fde3aa115693e76c2cce56233518d3e52af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Statistical Reproducibility Verification

## Summary

Reproduces published statistical results — here, fdr2d rejection counts from the MetPC biomarker-discovery procedure applied to a 3226 x 57 schisandra metabolomics matrix (Korea vs China) — by re-running the analysis in R and validating computed counts against reported values.

## When to use

Use when a published finding reports concrete numerical outputs (e.g. rejection counts at fdr2d levels 0.05, 0.1, 0.2, 0.3) that must be re-derived from raw processed data to confirm reproducibility.

## When NOT to use

- Input is raw, unprocessed spectral data rather than the preprocessed 3226 x 57 feature matrix — preprocessing is a separate pipeline step
- The goal is de novo biomarker discovery rather than validating a specific published numerical result against reported counts
- The comparison involves more than the two groups (Korea vs China) for which the fdr2d rejection counts were reported

## Inputs

- Processed metabolomics data matrix: 3226 metabolite features x 57 samples (schisandra Korea-vs-China contrast)
- Published target values: fdr2d rejection counts of 1092, 1326, 1782, 2120 at levels 0.05, 0.1, 0.2, 0.3

## Outputs

- fdr2d_rejections.csv containing rejected-hypothesis counts at fdr2d levels 0.05, 0.1, 0.2, and 0.3

## How to apply

Load the processed metabolomics data matrix (3226 metabolite features x 57 samples) into R and identify the two-group contrast (Korea vs China) to test. Apply the two-dimensional local FDR (fdr2d) procedure from the MetPC biomarker-discovery step to all 3226 metabolite null hypotheses simultaneously, as described in the Metabolites 2019 MetPC paper (DOI 10.3390/metabo9050103). Compute the number of rejections at each fdr2d cutoff of 0.05, 0.1, 0.2, and 0.3 using the paper's fdr2d thresholding rule. Write the per-cutoff rejected-hypothesis counts to fdr2d_rejections.csv. The rationale is that reproduction is judged by exact numerical agreement: the computed counts must match the published values of 1092 (level 0.05), 1326 (0.1), 1782 (0.2), and 2120 (0.3).

## Related tools

- **MetPC** (Provides the biomarker-discovery step implementing the two-dimensional local FDR (fdr2d) procedure used to compute rejections across the four cutoff levels) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript reproduce_fdr2d_rejections.R --input schisandra_processed_3226x57.csv --group1 Korea --group2 China --levels 0.05,0.1,0.2,0.3 --out fdr2d_rejections.csv
```

## Evaluation signals

- Computed rejection count at fdr2d level 0.05 equals 1092
- Computed rejection counts at fdr2d levels 0.1, 0.2, and 0.3 equal 1326, 1782, and 2120 respectively
- Output file fdr2d_rejections.csv exists and contains one count per cutoff level
- The analysis was run on exactly 3226 metabolite hypotheses (matching the input matrix dimensionality)
- The same two-group contrast (Korea vs China) and fdr2d thresholding rule as the source study were used

## Limitations

- Exact count reproduction depends on the identically preprocessed 3226 x 57 data matrix; differences in upstream preprocessing will change fdr2d outputs
- The fdr2d procedure controls two-dimensional FDR only when testing multiple hypotheses simultaneously, so it cannot be validated on single-hypothesis tests
- No discussion-section text was extractable from the source, so additional caveats on method sensitivity could not be captured

## Evidence

- [other] Findings: [evidence span withheld — anchor status: not_found]
- [other] Workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
