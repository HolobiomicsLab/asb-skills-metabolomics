---
name: roc-auc-ranking-analysis
description: Use when you have per-feature ROC AUC values (e.g. one AUC per metabolite
  with 95% CIs from an OPLS-DA model) and need to verify that a multivariate pattern
  or panel outperforms every individual feature.
when_to_use_negative:
- Input is raw GCxGC-TOFMS spectra or peak tables rather than precomputed per-metabolite
  AUC values — classification modeling (OPLS-DA, ROC curves) must come first.
- The goal is separating S. Typhi from S. Paratyphi A carriers rather than combined
  Salmonella carriage vs non-carriage controls; the 0.935 and >= 0.95 benchmarks apply
  to the combined-carriage comparison.
- No multivariate benchmark (pattern or panel AUC) exists to compare against, so the
  ranking cannot test the outperformance claim.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    instrument that produced the 195-metabolite plasma profiles from which the AUC
    values are derived
- name: pandas
  role: Used to rank the 195 metabolites in descending order of AUC and to append
    benchmark rows and the verification flag
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:2778c8a04c70bd8fbf23fdd589f3724d44212a0c61a56ddb24bb4bb33a91aba4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# roc-auc-ranking-analysis

## Summary

A benchmarking procedure that ranks per-feature ROC AUC values against a multivariate-pattern AUC to verify that a combined metabolite pattern outperforms any single metabolite. In this paper, it tests whether any of 195 GCxGC-TOFMS metabolites matches the OPLS-DA carriage-pattern AUC.

## When to use

Use when you have per-feature ROC AUC values (e.g. one AUC per metabolite with 95% CIs from an OPLS-DA model) and need to verify that a multivariate pattern or panel outperforms every individual feature.

## When NOT to use

- Input is raw GCxGC-TOFMS spectra or peak tables rather than precomputed per-metabolite AUC values — classification modeling (OPLS-DA, ROC curves) must come first.
- The goal is separating S. Typhi from S. Paratyphi A carriers rather than combined Salmonella carriage vs non-carriage controls; the 0.935 and >= 0.95 benchmarks apply to the combined-carriage comparison.
- No multivariate benchmark (pattern or panel AUC) exists to compare against, so the ranking cannot test the outperformance claim.

## Inputs

- S2_Table.docx containing metabolite identifiers, AUC values, and 95% confidence intervals for 195 metabolites from the OPLS-DA model
- Reference benchmark AUCs: Fig 2A multivariate pattern AUC (>= 0.95) and five-metabolite panel AUC (0.935)
- Five named panel-metabolite AUCs (hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876) for cross-checking

## Outputs

- ranked_metabolite_auc_table.csv — 195 metabolites sorted by descending AUC with benchmark rows and a boolean verification flag
- Boolean verification flag stating whether max individual AUC < 0.935 and < pattern AUC

## How to apply

Parse the supplementary per-feature table (S2_Table.docx) to extract metabolite identifiers, AUC values, and 95% confidence intervals computed from relative concentrations for all 195 metabolites in the OPLS-DA model. Rank the metabolites in descending AUC order using pandas, then append reference benchmark rows: the combined-carriage OPLS-DA pattern AUC (>= 0.95, from Fig 2A) and the five-metabolite panel AUC of 0.935. Cross-check the five panel metabolites (hexanoic acid AUC 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876) against S2 Fig before comparing. Record a boolean flag verifying that the maximum individual-metabolite AUC is strictly less than 0.935 and less than the Fig 2A pattern AUC, and write the ranked table with benchmarks to CSV.

## Related tools

- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry instrument that produced the 195-metabolite plasma profiles from which the AUC values are derived)
- **pandas** (Used to rank the 195 metabolites in descending order of AUC and to append benchmark rows and the verification flag)

## Examples

```
import pandas as pd; s2 = pd.read_table('S2_Table.txt'); ranked = s2.sort_values('AUC', ascending=False); ranked.to_csv('ranked_metabolite_auc_table.csv', index=False)
```

## Evaluation signals

- Maximum individual-metabolite AUC in the ranked table is strictly less than 0.935 (the five-metabolite panel AUC).
- Maximum individual-metabolite AUC is less than the Fig 2A multivariate pattern AUC (>= 0.95), confirming the paper's claim.
- The five panel metabolites recover exactly when cross-checked: hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876.
- Ranked table contains all 195 metabolites plus two benchmark rows, and the boolean verification flag is True.
- AUC values and 95% confidence intervals in the output match the S2_Table.docx source exactly (no truncation of metabolites or CI columns).

## Limitations

- The study had a small sample size, which constrains the reliability of the AUC estimates being ranked.
- There was no independent validation cohort: 'resulting metabolite profiles should be validated in an additional, independent cohort. However, this was not possible in this setting'.
- All patients presented with gallbladder conditions, so underlying disease may confound the metabolite signatures underlying the AUCs.

## Evidence

- [supplementary] No individual metabolite among the 195 analyzed had an AUC value as high as or higher than the AUC value of the multivariate metabolite pattern: [evidence span withheld — anchor status: not_found]
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [intro] we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage: "we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage"
- [supplementary] Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
