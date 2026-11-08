---
name: immunohistochemistry-scoring-extraction
description: Use when extracting IHC scoring parameters, k-means clustering subgroups,
  or tumor counts/percentages for phosphoAKT1/MYC stratification from prostate cancer
  cohort articles.
when_to_use_negative:
- The article describes western blot or immunoblot band densitometry without IHC scoring
  or cell-level counts.
- The stratification markers are not phosphoAKT1 and MYC (e.g. other IHC panels with
  different markers).
- No k-means clustering of densitometric values is reported and no subgroup counts/percentages
  are extractable.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: GraphPad Prism
  role: Used for Fisher Test statistical evaluation of IHC stratification associations
    at p<0.05 significance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:f06f4a40d12fbcace09611ffe8dc1c30ec5b1f7683bbb3dd1a0015546bcd5123
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# immunohistochemistry-scoring-extraction

## Summary

Extracts stratification parameters and subgroup counts from phosphoAKT1/MYC immunohistochemistry and k-means clustering of densitometric values for a 60-tumor pathological stage T2 prostate cancer cohort.

## When to use

Use when extracting IHC scoring parameters, k-means clustering subgroups, or tumor counts/percentages for phosphoAKT1/MYC stratification from prostate cancer cohort articles.

## When NOT to use

- The article describes western blot or immunoblot band densitometry without IHC scoring or cell-level counts.
- The stratification markers are not phosphoAKT1 and MYC (e.g. other IHC panels with different markers).
- No k-means clustering of densitometric values is reported and no subgroup counts/percentages are extractable.

## Inputs

- Article text with Methods immunohistochemistry passage
- Article text with Results k-means clustering passage
- Article text with cohort description (60 T2 tumors)
- Source cards for the article

## Outputs

- TSV table of IHC scoring parameters and subgroup counts/percentages
- Stratification table stratification_table.tsv

## How to apply

Read the Methods immunohistochemistry passage and extract the markers scored (MYC, stathmin, GLUT-1, phosphoAKT1), the 50% positive-cell cutoff, and the Fisher Test statistic computed via GraphPad Prism 5.0 with p<0.05 significance. Read the Results k-means clustering passage and record the four subgroups with their counts and percentages: phosphoAKT1-high/MYC-high 6/60 (10%), phosphoAKT1-high/MYC-low 21/60 (35%), phosphoAKT1-low/MYC-high 9/60 (15%), and phosphoAKT1-low/MYC-low 24/60 (40%). Record cohort context (60 T2 tumors; 22 high Gleason, 38 low Gleason) and IHC scoring detail (entire section evaluated, 5 representative areas at 20x, average of 500 cells per section). Assemble these into a table with columns for subgroup, count, percentage, and source statement, then verify each value matches verbatim statements.

## Related tools

- **GraphPad Prism** (Used for Fisher Test statistical evaluation of IHC stratification associations at p<0.05 significance)

## Examples

```
python extract_stratification_table.py --input article_methods_results.txt --output stratification_table.tsv
```

## Evaluation signals

- All four subgroup counts sum to 60 and percentages sum to 100% (10+35+15+40).
- Each extracted parameter and count matches verbatim statements in the article text.
- Markers scored (MYC, stathmin, GLUT-1, phosphoAKT1) and the 50% positive-cell cutoff are present.
- Fisher Test via GraphPad Prism 5.0 with p<0.05 significance is recorded.

## Limitations

- The phosphoAKT1/MYC stratification is independent of Gleason grade and pathologic staging; a separate k-means or Silhouette analysis of human tumors found 3 clusters where the phosphoAKT1-low/MYC-high and phosphoAKT1-high/MYC-low tumors differentially segregated (Fisher test, p<0.01).
- Extraction depends on the article explicitly stating subgroup counts, percentages, and cutoffs; values not present in the text cannot be recovered.

## Evidence

- [methods] Immunohistochemistry markers scored: [evidence span withheld — anchor status: not_found]
- [results] Cohort description: [evidence span withheld — anchor status: not_found]
