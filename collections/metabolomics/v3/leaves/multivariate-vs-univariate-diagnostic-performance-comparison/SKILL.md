---
name: multivariate-vs-univariate-diagnostic-performance-comparison
description: Use when a supervised multivariate model (e.g. OPLS-DA) is claimed to
  outperform single markers, and per-feature AUCs plus the model-level AUC are available
  for a head-to-head ROC comparison.
when_to_use_negative:
- The input is already a ranked/filtered feature table without per-feature AUC values
  and confidence intervals, so no ROC comparison can be recomputed.
- The goal is to compare two multivariate models (e.g. OPLS-DA vs PCA) rather than
  one multivariate pattern against individual markers.
- AUCs were computed on different sample sets or cross-validation schemes, making
  the individual-vs-pattern AUC comparison invalid.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    that generated the plasma metabolite data underlying the 195 metabolites and their
    AUCs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:9c1ec4cc3bdf217a26ab2be7b380dd9131e861c224dbf65bba3df37906ff807f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multivariate-vs-univariate-diagnostic-performance-comparison

## Summary

Benchmarks ROC AUCs of individual metabolites (from S2 Table) against the multivariate OPLS-DA latent biomarker pattern AUC to verify that a co-varying metabolite pattern outperforms any single metabolite as a diagnostic classifier.

## When to use

Use when a supervised multivariate model (e.g. OPLS-DA) is claimed to outperform single markers, and per-feature AUCs plus the model-level AUC are available for a head-to-head ROC comparison.

## When NOT to use

- The input is already a ranked/filtered feature table without per-feature AUC values and confidence intervals, so no ROC comparison can be recomputed.
- The goal is to compare two multivariate models (e.g. OPLS-DA vs PCA) rather than one multivariate pattern against individual markers.
- AUCs were computed on different sample sets or cross-validation schemes, making the individual-vs-pattern AUC comparison invalid.

## Inputs

- S2_Table.docx supplementary file with AUC values and 95% confidence intervals for 195 metabolites from GCxGC-TOFMS relative concentrations
- Reference benchmark AUCs: OPLS-DA pattern AUC from Fig 2A and five-metabolite panel AUC (0.935)

## Outputs

- ranked_metabolite_auc_table.csv with 195 metabolites ranked by descending AUC, benchmark rows, and a boolean verification flag

## How to apply

Load the S2_Table.docx supplementary file and parse metabolite identifiers, AUC values, and 95% confidence intervals computed from relative concentrations for all 195 metabolites in the OPLS-DA model. Rank the metabolites in descending AUC with pandas, then append benchmark rows: the combined-carriage OPLS-DA pattern AUC from Fig 2A (>= 0.95) and the five-metabolite panel AUC of 0.935 (hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876, cross-checked against S2 Fig). Verify that the maximum individual-metabolite AUC is strictly below both benchmarks, record a boolean flag for the paper's claim, and write ranked_metabolite_auc_table.csv. The rationale is the paper's hypothesis that a combination of co-varying metabolites yields more sensitive and specific biomarkers of typhoid carriage than single metabolites.

## Related tools

- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry that generated the plasma metabolite data underlying the 195 metabolites and their AUCs)

## Examples

```
df = pd.read_excel('S2_Table.docx', engine=None); df.sort_values('AUC', ascending=False).to_csv('ranked_metabolite_auc_table.csv', index=False)
```

## Evaluation signals

- The maximum individual-metabolite AUC in ranked_metabolite_auc_table.csv must be strictly less than 0.935 (the five-metabolite panel AUC).
- The maximum individual-metabolite AUC must be strictly less than the Fig 2A OPLS-DA pattern AUC (>= 0.95).
- The table must contain exactly 195 metabolite rows from the OPLS-DA model, each with an AUC and 95% CI.
- The five named panel metabolites (hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876) must match S2 Fig values.
- A boolean verification flag confirming the paper's claim must be present in the output CSV.

## Limitations

- Small sample size was the major study limitation, making AUC estimates imprecise and hard to overcome given ethical, surgical, and diagnostic constraints.
- No independent validation cohort existed; resulting metabolite profiles should be validated in an additional, independent cohort.
- All patients had gallbladder conditions, so the cholecystectomy indication may confound the metabolite signatures used in the AUC comparison.

## Evidence

- [intro] we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage: "we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage"
- [supplementary] No individual metabolite among the 195 analyzed had an AUC value as high as or higher than the AUC value of the multivariate metabolite pattern: [evidence span withheld — anchor status: not_found]
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] The major limitation of this study was the small sample size: "The major limitation of this study was the small sample size"
