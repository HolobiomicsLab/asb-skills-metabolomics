---
name: gleason-grade-clinical-annotation-handling
description: Use when tumor samples carry both a binary molecular subgroup assignment
  (e.g. AKT1-high/low, MYC-high/low) and a clinical grade/stage label, and the goal
  is to test independence of the molecular status from the clinical variable.
when_to_use_negative:
- The goal is to test association of AKT1/MYC status with metabolomic cluster membership
  rather than clinical grade — in that context the Fisher test was significant (p<0.01),
  not a null result.
- Grade labels are graded (e.g. continuous Gleason score) rather than the paper's
  binary high (4+3/4+4) vs low (3+3/3+4) split; a rank or trend test would be needed.
- The molecular subgroup is not binary — the 4-group k-means assignment requires the
  2x4 or collapsed 2x2 formulation before a Fisher test applies.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GraphPad Prism
  role: Fisher's exact test on 2x2 contingency tables of phosphoAKT1/MYC status vs
    Gleason grade (paper reports Prism 5.0 Fisher Test)
- name: scipy.stats.fisher_exact
  role: Python equivalent for reproducing the paper's Fisher tests on AKT1-vs-grade
    and MYC-vs-grade tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:7f692ff7b3bdc8e7cb49f6f45cb809865228bd5e60c983928c8bf991fb903f21
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gleason-grade-clinical-annotation-handling

## Summary

Statistical test of whether a molecular subtype assignment (here phosphoAKT1/MYC k-means subgroups) is independent of a clinical annotation (Gleason grade, pathologic stage) using Fisher's exact tests on 2x2 contingency tables with p<0.05 as the non-association criterion.

## When to use

Use when tumor samples carry both a binary molecular subgroup assignment (e.g. AKT1-high/low, MYC-high/low) and a clinical grade/stage label, and the goal is to test independence of the molecular status from the clinical variable.

## When NOT to use

- The goal is to test association of AKT1/MYC status with metabolomic cluster membership rather than clinical grade — in that context the Fisher test was significant (p<0.01), not a null result.
- Grade labels are graded (e.g. continuous Gleason score) rather than the paper's binary high (4+3/4+4) vs low (3+3/3+4) split; a rank or trend test would be needed.
- The molecular subgroup is not binary — the 4-group k-means assignment requires the 2x4 or collapsed 2x2 formulation before a Fisher test applies.

## Inputs

- Cohort composition table: 60 T2 tumors, 22 high Gleason (4+3/4+4) vs 38 low Gleason (3+3/3+4)
- K-means subgroup counts from phosphoAKT1/MYC immunoblot densitometry: 6/60, 21/60, 9/60, 24/60
- Per-sample binary molecular status labels (phosphoAKT1-high/low, MYC-high/low)
- Per-sample Gleason grade and pathologic stage (T2) annotations

## Outputs

- 2x2 contingency tables (AKT1 status vs Gleason grade; MYC status vs Gleason grade) and a 2x4 table
- Fisher's exact test p-values and odds ratios for each molecular-status vs grade comparison
- Independence verdict: all p > 0.05 reproducing the reported AKT1/MYC–Gleason independence (Fig. S1A)

## How to apply

Load the cohort composition as reported (here: 60 pathologic stage T2 prostate tumors split into 22 high Gleason — Gleason 4+3 or 4+4 — and 38 low Gleason — 3+3 or 3+4, with k-means subgroups phosphoAKT1-high/MYC-high = 6, phosphoAKT1-high/MYC-low = 21, phosphoAKT1-low/MYC-high = 9, phosphoAKT1-low/MYC-low = 24). Construct 2x2 and 2x4 contingency tables crossing AKT1-high vs AKT1-low and MYC-high vs MYC-low against high vs low Gleason grade, verifying that all margins sum to 60. Run Fisher's exact tests on each AKT1-vs-grade and MYC-vs-grade table — the paper used GraphPad Prism 5.0 Fisher Test, reproducible with scipy.stats.fisher_exact — applying the p<0.05 significance threshold. A correct application reproduces the paper's conclusion that phosphoAKT1 and MYC levels were not associated with Gleason grade (Fig. S1A): every p-value must exceed 0.05, demonstrating the AKT1/MYC metabolic phenotypes are independent of tumor grade. Note the contrast: the same Fisher-test machinery detected significant differential segregation (p<0.01) of phosphoAKT1-low/MYC-high vs phosphoAKT1-high/MYC-low tumors across metabolomic clusters, so the null result is grade-specific, not a power artifact.

## Related tools

- **GraphPad Prism** (Fisher's exact test on 2x2 contingency tables of phosphoAKT1/MYC status vs Gleason grade (paper reports Prism 5.0 Fisher Test))
- **scipy.stats.fisher_exact** (Python equivalent for reproducing the paper's Fisher tests on AKT1-vs-grade and MYC-vs-grade tables)

## Examples

```
from scipy.stats import fisher_exact; _, p = fisher_exact([[akt1high_gradehigh, akt1high_gradelow], [akt1low_gradehigh, akt1low_gradelow]]); assert p > 0.05
```

## Evaluation signals

- All Fisher-test p-values for AKT1/MYC status vs Gleason grade exceed 0.05, matching the reported independence (Fig. S1A).
- Contingency-table margins sum to 60 (22 high Gleason + 38 low Gleason; subgroup counts 6+21+9+24 = 60).
- Subgroup percentages match the reported 10% (6/60), 35% (21/60), 15% (9/60), 40% (24/60).
- The same test machinery yields p<0.01 for the known positive contrast (phosphoAKT1-low/MYC-high vs phosphoAKT1-high/MYC-low cluster segregation), confirming the test is functioning.

## Limitations

- Independence was tested only within the 60 pathologic stage T2 tumors; extension to other stages is not supported by this analysis.
- Gleason grade was dichotomized (4+3/4+4 vs 3+3/3+4), which discards within-category variation.
- A null Fisher result with small subgroup counts (e.g. 6/60) has limited power; absence of association is not evidence of equivalence.

## Evidence

- [abstract] the AKT1/MYC status was independent of Gleason grade and pathologic staging: "the AKT1/MYC status was independent of Gleason grade and pathologic staging"
