---
name: per-year-performance-table-generation
description: Use when reproducing a baseline comparison across multiple harvest years
  (2001/2003/2004) where a trained baseline (CHESHIRE, threshold=0.5) must be tabulated
  against reported method results (NetAurHPD) per training-fraction session.
when_to_use_negative:
- The task needs NetAurHPD's own pipeline execution (auralization, M5 training, Youden
  threshold) rather than tabulation of comparison results
- There is only a single training run per dataset with no per-year split or training-fraction
  sessions, so a Figure-5-style year-by-fraction table cannot be formed
- Evaluation data are not the tough-condition pathway test sets (40 TomatoCyc / 27
  MetaCyc / 13 random) but standard full-training cross-validation folds
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CHESHIRE
  role: State-of-the-art hyperlink prediction baseline trained per year and training
    fraction (75 epochs, threshold 0.5) whose AUC and central-pathway results populate
    the table
- name: PyTorch
  role: Deep-learning framework used to train the CHESHIRE baseline sessions whose
    outputs are tabulated
- name: NetAurHPD
  role: Provides the reference results (NetAurHPD and NetAurHPD-with-one-Augmentation
    from Figure 5) that CHESHIRE rows are compared against
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:837ff88d36b0060369f0bd03c41f9bacb3460b25998e9f7db435eea49832036a
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - CHESHIRE
  - NetAurHPD
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# per-year-performance-table-generation

## Summary

Aggregates CHESHIRE baseline results (AUC and correctly classified central pathways) per tomato-pericarp year and training fraction into a comparison table against reported NetAurHPD and NetAurHPD-with-one-Augmentation values, reproducing Figure 5's left/right panels.

## When to use

Use when reproducing a baseline comparison across multiple harvest years (2001/2003/2004) where a trained baseline (CHESHIRE, threshold=0.5) must be tabulated against reported method results (NetAurHPD) per training-fraction session.

## When NOT to use

- The task needs NetAurHPD's own pipeline execution (auralization, M5 training, Youden threshold) rather than tabulation of comparison results
- There is only a single training run per dataset with no per-year split or training-fraction sessions, so a Figure-5-style year-by-fraction table cannot be formed
- Evaluation data are not the tough-condition pathway test sets (40 TomatoCyc / 27 MetaCyc / 13 random) but standard full-training cross-validation folds

## Inputs

- Tomato pericarp metabolite correlation networks for 2001, 2003, 2004 (75/75/78 metabolite nodes with 473/869/338 edges)
- Labeled pathway pools per year: 129 TomatoCyc + 65 MetaCyc + 64 random training pathways and test sets of 40 TomatoCyc + 27 MetaCyc + 13 random pathways
- Per-session CHESHIRE evaluation records (year, training fraction, AUC, count of correctly classified central pathways)
- Reported NetAurHPD and NetAurHPD-with-one-Augmentation AUC values from Figure 5 for side-by-side comparison

## Outputs

- Per-year, per-training-fraction performance table of CHESHIRE AUC values (10 sessions per year, 2001/2003/2004)
- Per-year counts of correctly classified central pathways (13 critical TomatoCyc pathways) for CHESHIRE
- Combined baseline-vs-NetAurHPD comparison table reproducing Figure 5's left (AUC) and right (central pathways) panels

## How to apply

Train the CHESHIRE baseline per year on the reduced pathway training sets (10 sessions per year, training fraction growing 10% to 100%, 75 epochs, threshold 0.5), then evaluate on each year's tough-condition test set (40 TomatoCyc, 27 MetaCyc, 13 random pathways) recording AUC and the count of correctly classified central pathways (13 TomatoCyc pathways critical for organism survival). Build one row per (year, session, training fraction) with micro-average AUC and central-pathway count, and place CHESHIRE values alongside the reported NetAurHPD and NetAurHPD-with-one-Augmentation values from Figure 5. Use the resulting table to diagnose where CHESHIRE fails to reach high AUC and mispredicts central pathways (2001, 2003) versus where it is competitive (2004). Validate by confirming the reproduced curves and counts match Figure 5's left and right panels for all three years.

## Related tools

- **CHESHIRE** (State-of-the-art hyperlink prediction baseline trained per year and training fraction (75 epochs, threshold 0.5) whose AUC and central-pathway results populate the table)
- **PyTorch** (Deep-learning framework used to train the CHESHIRE baseline sessions whose outputs are tabulated)
- **NetAurHPD** (Provides the reference results (NetAurHPD and NetAurHPD-with-one-Augmentation from Figure 5) that CHESHIRE rows are compared against) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
cheshire_df.groupby(['year','train_frac'])[['auc','central_correct']].mean().reset_index().to_csv('fig5_cheshire_vs_netaurhpd_table.csv', index=False)
```

## Evaluation signals

- Reproduced CHESHIRE AUC curves per training fraction match the left panels of Figure 5 for 2001, 2003, and 2004
- Reproduced central-pathway counts match the right panels of Figure 5, with NetAurHPD correctly predicting more central pathways than CHESHIRE in 2001 and 2003
- NetAurHPD outperforms CHESHIRE in general AUC across all three years in the tabulated comparison
- Table covers exactly 10 sessions per year spanning training fractions from 10% to 100%
- NetAurHPD micro-average AUC anchors (0.87 for 2001, 0.857 for 2003, 0.884 for 2004) fall within the expected comparison range against CHESHIRE rows

## Limitations

- Models do not consistently improve with more data — in some sessions the AUCs of all models decreased, so table rows are not monotone in training fraction
- CHESHIRE struggles to reach high AUC and yields poor central-pathway predictions under tough realistic conditions (limited known pathways) in 2001 and 2003, so baseline rows may look uniformly weak
- Per-session variability is non-trivial (NetAurHPD alone shows an average AUC range of 0.082), so single-session entries should not be treated as stable estimates

## Evidence

- [methods] Reference micro-average AUC anchors for the tabulated years: [evidence span withheld — anchor status: not_found]
