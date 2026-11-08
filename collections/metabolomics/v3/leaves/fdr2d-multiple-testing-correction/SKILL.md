---
name: fdr2d-multiple-testing-correction
description: Use when a metabolomics study (e.g. the 3226 x 57 schisandra Korea-vs-China
  matrix) requires simultaneous testing of thousands of metabolite null hypotheses
  under two-dimensional FDR control at levels such as 0.05, 0.1, 0.2, or 0.3.
when_to_use_negative:
- When only a small number of hypotheses are tested and single-hypothesis testing
  or a one-dimensional FDR method (e.g. Benjamini-Hochberg) is appropriate instead
  of fdr2d
- When the input is raw instrument data that has not yet been preprocessed and metabolite-identified
  by the upstream MetPC pipeline steps
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetPC
  role: Provides the biomarker-discovery procedure that controls the two-dimensional
    false discovery rate (fdr2d) when testing multiple metabolite hypotheses simultaneously;
    used to compute fdr2d rejections on the 3226 x 57 schisandra matrix.
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:1549721e84494756274a29fbf6645b727b0f2db628fc2135ec3cfdfcaff9c261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fdr2d-multiple-testing-correction

## Summary

Applies the two-dimensional local FDR (fdr2d) multiple-testing procedure from the MetPC biomarker-discovery step to simultaneously test thousands of metabolite hypotheses while controlling fdr2d, yielding rejection counts at specified fdr2d cutoffs.

## When to use

Use when a metabolomics study (e.g. the 3226 x 57 schisandra Korea-vs-China matrix) requires simultaneous testing of thousands of metabolite null hypotheses under two-dimensional FDR control at levels such as 0.05, 0.1, 0.2, or 0.3.

## When NOT to use

- When only a small number of hypotheses are tested and single-hypothesis testing or a one-dimensional FDR method (e.g. Benjamini-Hochberg) is appropriate instead of fdr2d
- When the input is raw instrument data that has not yet been preprocessed and metabolite-identified by the upstream MetPC pipeline steps

## Inputs

- Processed metabolomics data matrix (3226 metabolite features x 57 samples) for a two-group (Korea vs China) comparison
- Vector of fdr2d cutoff levels (0.05, 0.1, 0.2, 0.3)

## Outputs

- Rejected-hypothesis counts per fdr2d cutoff level (e.g. fdr2d_rejections.csv)
- Set of rejected metabolite features per cutoff

## How to apply

Load the processed metabolomics data matrix (e.g. 3226 metabolite features x 57 samples) into R and treat each metabolite feature as one null hypothesis in a Korea-vs-China two-group comparison. Apply the fdr2d procedure from the MetPC biomarker-discovery stage, which controls the two-dimensional false discovery rate when testing all hypotheses simultaneously rather than one feature at a time. Compute the number of rejected hypotheses using the fdr2d thresholding rule at each of the four cutoff levels 0.05, 0.1, 0.2, and 0.3, and write the per-cutoff rejection counts to a CSV (e.g. fdr2d_rejections.csv). The counts should scale monotonically with the cutoff, as larger fdr2d levels reject more hypotheses.

## Related tools

- **MetPC** (Provides the biomarker-discovery procedure that controls the two-dimensional false discovery rate (fdr2d) when testing multiple metabolite hypotheses simultaneously; used to compute fdr2d rejections on the 3226 x 57 schisandra matrix.) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript fdr2d_rejections.R --input schisandra_processed.csv --groups Korea,China --levels 0.05,0.1,0.2,0.3 --output fdr2d_rejections.csv
```

## Evaluation signals

- Rejection counts must be monotonically non-decreasing as the fdr2d level increases from 0.05 to 0.3
- On the schisandra Korea-vs-China 3226-feature data, rejections must equal 1092 at fdr2d level 0.05 and 1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3, respectively
- The number of rejected hypotheses at any cutoff must not exceed 3226 (the total number of metabolite hypotheses tested)
- Output CSV fdr2d_rejections.csv must contain one count per requested cutoff level (0.05, 0.1, 0.2, 0.3)

## Limitations

- Expected rejection counts are tied to the specific 3226 x 57 schisandra Korea-vs-China dataset; other datasets will produce different counts at the same fdr2d levels
- Requires the input matrix to be the processed feature matrix from upstream MetPC preprocessing; raw or unprocessed data is not a valid input

## Evidence

- [abstract] Biomarker discovery under fdr2d control: "For biomarker metabolite discovery, our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [results] Rejection counts on the schisandra dataset: [evidence span withheld — anchor status: not_found]
- [methods] Data matrix dimensions and grouping: [evidence span withheld — anchor status: not_found]
