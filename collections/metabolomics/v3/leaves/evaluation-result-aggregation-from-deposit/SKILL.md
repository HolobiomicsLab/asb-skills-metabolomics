---
name: evaluation-result-aggregation-from-deposit
description: Use when two annotation pipelines have been evaluated on the same query
  spectra and you need per-method correctness counts, overlap statistics, and a figure
  showing which correct annotations are unique to each method.
when_to_use_negative:
- Only one annotation method has been run, so no cross-method overlap can be computed
  — use single-method evaluation metrics instead
- Correctness labels are graded (e.g. ranked candidate lists) rather than Boolean
  per-query labels — aggregation into a 2x2 contingency table is not appropriate
- The two methods were evaluated on different query sets with no shared query spectra
  to intersect
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: BAM
  role: Biotransformation-based annotation method whose evaluation results (correct
    query labels) are one input to the overlap analysis
  repo: https://github.com/HassounLab/BAM
- name: SIRIUS 5.6.3
  role: Baseline molecular formula and structure annotation method whose correctness
    labels are the comparison input
- name: Python 3.9
  role: Environment for loading, crossing, and counting correctness labels
- name: pandas / scikit-learn
  role: Used to build the contingency table and compute counts of queries correct
    in both, only in BAM, and only in SIRIUS
- name: Matplotlib / Seaborn
  role: Produce the bar or Venn-style uniqueness breakdown figure saved as PNG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:88283aad0f202437eec05be06ef3fba072d885200ff15efe62ab35aee5178bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - Matplotlib
  - Matplotlib / Seaborn
  - NumPy
  - PROXIMAL2
  - Pandas
  - Python 3.9
  - SIRIUS 5.6.3
  - Seaborn
  - pandas / scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# evaluation-result-aggregation-from-deposit

## Summary

Aggregate per-query Boolean correctness labels from two annotation methods (BAM and SIRIUS 5.6.3) run on the same 60,368 anchor–suspect pair data set into a contingency table and uniqueness breakdown that demonstrates complementary annotation results.

## When to use

Use when two annotation pipelines have been evaluated on the same query spectra and you need per-method correctness counts, overlap statistics, and a figure showing which correct annotations are unique to each method.

## When NOT to use

- Only one annotation method has been run, so no cross-method overlap can be computed — use single-method evaluation metrics instead
- Correctness labels are graded (e.g. ranked candidate lists) rather than Boolean per-query labels — aggregation into a 2x2 contingency table is not appropriate
- The two methods were evaluated on different query sets with no shared query spectra to intersect

## Inputs

- BAM evaluation results with per-query Boolean correctness labels (from Zenodo deposit 10.5281/zenodo.13903946)
- SIRIUS 5.6.3 baseline results (molecular formula annotation followed by molecular structure annotation) on the same 60,368 anchor–suspect pairs
- Shared query spectrum identifiers linking both result sets

## Outputs

- Contingency table of BAM-correct vs SIRIUS-correct queries (CSV)
- Uniqueness breakdown figure (PNG) showing counts of queries correct in both, only in BAM, and only in SIRIUS
- Overlap statistics supporting or refuting the complementarity claim

## How to apply

Load BAM evaluation results from the Zenodo deposit (DOI 10.5281/zenodo.13903946) together with the SIRIUS 5.6.3 baseline results generated on the same 60,368 anchor–suspect pair data set, keeping for each query spectrum a Boolean correctness label per method. Restrict to query spectra shared by both runs and build a contingency table crossing BAM correctness against SIRIUS correctness using pandas/scikit-learn in the Python 3.9 environment. Compute three counts: queries correct in both methods, correct only in BAM, and correct only in SIRIUS, since the paper's claim rests on the majority of correct queries being unique to each method. Plot the uniqueness breakdown (bar or Venn-style) with Matplotlib/Seaborn, save the contingency table as CSV and the figure as PNG, and verify the overlap pattern supports the complementarity claim.

## Related tools

- **BAM** (Biotransformation-based annotation method whose evaluation results (correct query labels) are one input to the overlap analysis) — https://github.com/HassounLab/BAM
- **SIRIUS 5.6.3** (Baseline molecular formula and structure annotation method whose correctness labels are the comparison input)
- **Python 3.9** (Environment for loading, crossing, and counting correctness labels)
- **pandas / scikit-learn** (Used to build the contingency table and compute counts of queries correct in both, only in BAM, and only in SIRIUS)
- **Matplotlib / Seaborn** (Produce the bar or Venn-style uniqueness breakdown figure saved as PNG)

## Examples

```
import pandas as pd; df = pd.read_csv('bam_results.csv').merge(pd.read_csv('sirius_results.csv'), on='query_id'); ct = pd.crosstab(df.bam_correct, df.sirius_correct); ct.to_csv('contingency.csv')
```

## Evaluation signals

- Total queries in the contingency table equals the number of query spectra shared by both the BAM and SIRIUS runs (60,368 anchor–suspect pairs in the paper)
- Counts of both-correct, BAM-only-correct, and SIRIUS-only-correct sum to the total number of correctly annotated queries across methods
- The uniqueness breakdown shows the majority of correct queries are unique to each method, matching the paper's complementarity finding
- The CSV contingency table row/column sums match the per-method correct totals (BAM alone: 24.2% of examined anchor–suspect cases)
- PNG figure and CSV table are produced and mutually consistent

## Limitations

- Comparison requires both methods to be evaluated on the same 60,368 anchor–suspect pair data set; results are not transferable to different query sets
- SIRIUS baseline followed a two-stage protocol where structure annotation was performed only if the correct molecular formula was generated, which conditions its correctness labels
- Correctness labels are Boolean per query; ties in candidate ranking were resolved by average rank, which affects what counts as 'correct'

## Evidence

- [other] When comparing correct queries from BAM and SIRIUS, the majority of correct queries are unique to each method, confirming that biotransformation-based annotation and SIRIUS provide complementary annotation results.: [evidence span withheld — anchor status: not_found]
- [methods] we applied SIRIUS (version 5.6.3) to the query spectra represented in the same data set of 60,368 anchor–suspect pairs: [evidence span withheld — anchor status: not_found]
- [methods] All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946): "All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946)"
- [methods] we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation.: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [abstract] BAM was able to assign correct molecular structures to 24.2% of examined anchor-suspect cases: "BAM was able to assign correct molecular structures to 24.2% of examined anchor-suspect cases"
