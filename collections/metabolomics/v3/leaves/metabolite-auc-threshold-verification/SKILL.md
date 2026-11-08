---
name: metabolite-auc-threshold-verification
description: Use when GCxGC-TOFMS metabolomics data has produced an OPLS-DA multivariate
  pattern AUC plus per-metabolite AUCs with 95% CIs, and you must verify that the
  multivariate pattern's ROC AUC is not matched or exceeded by any single metabolite.
when_to_use_negative:
- Input is already a validated single-biomarker AUC table with no multivariate OPLS-DA
  pattern AUC available to benchmark against
- The goal is class separation between S. Typhi and S. Paratyphi A carriage (use the
  two-component OPLS-DA discrimination analysis instead of single-metabolite AUC ranking)
- Sample groups are defined by something other than Salmonella carriage vs. non-carriage
  in cholecystectomy patients, so the 0.935 panel and ≥0.95 pattern benchmarks do
  not apply
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCxGC-TOFMS
  role: Produced the plasma metabolite measurements from which the 195 OPLS-DA metabolites
    and their AUC values were computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:6e85ec3ce7cfbd3e17faa65eca818f10d198b5bed602ad8f265f3f8d908fb3ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolite-auc-threshold-verification

## Summary

Verification skill that ranks the per-metabolite AUC values from the 195-metabolite OPLS-DA model (S2 Table) and checks that no individual metabolite matches or exceeds the multivariate metabolite-pattern AUC, supporting the paper's claim that the co-varying metabolite pattern outperforms any single biomarker.

## When to use

Use when GCxGC-TOFMS metabolomics data has produced an OPLS-DA multivariate pattern AUC plus per-metabolite AUCs with 95% CIs, and you must verify that the multivariate pattern's ROC AUC is not matched or exceeded by any single metabolite.

## When NOT to use

- Input is already a validated single-biomarker AUC table with no multivariate OPLS-DA pattern AUC available to benchmark against
- The goal is class separation between S. Typhi and S. Paratyphi A carriage (use the two-component OPLS-DA discrimination analysis instead of single-metabolite AUC ranking)
- Sample groups are defined by something other than Salmonella carriage vs. non-carriage in cholecystectomy patients, so the 0.935 panel and ≥0.95 pattern benchmarks do not apply

## Inputs

- S2_Table.docx supplementary file with AUC values and 95% CIs for 195 metabolites from the OPLS-DA model
- Fig 2A combined-carriage OPLS-DA pattern AUC benchmark
- S2 Fig five-metabolite panel ROC curves (hexanoic acid, Unknown_087, Unknown_118, glutaric acid, Unknown_399)

## Outputs

- ranked_metabolite_auc_table.csv — 195 metabolites ranked by descending AUC, with benchmark reference rows and a boolean claim-verification flag

## How to apply

Load the S2_Table.docx supplementary file and parse metabolite identifiers, relative-concentration-derived AUC values, and 95% confidence intervals for all 195 metabolites in the OPLS-DA model; rank them in descending AUC with pandas. Append benchmark rows: the combined-carriage OPLS-DA pattern AUC from Fig 2A (≥ 0.95) and the five-metabolite panel AUC of 0.935, cross-checking the five named metabolites (hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876) against S2 Fig. Compare the maximum individual-metabolite AUC against both benchmarks and record a boolean verification flag — the claim holds only if the top metabolite is strictly below 0.935 and below the Fig 2A pattern AUC. Write the ranked table with benchmarks and flag to ranked_metabolite_auc_table.csv.

## Related tools

- **GCxGC-TOFMS** (Produced the plasma metabolite measurements from which the 195 OPLS-DA metabolites and their AUC values were computed)

## Examples

```
import pandas as pd; df = pd.read_excel('S2_Table.docx'); ranked = df.sort_values('AUC', ascending=False); max_auc = ranked['AUC'].max(); claim_holds = max_auc < 0.935; ranked.to_csv('ranked_metabolite_auc_table.csv')
```

## Evaluation signals

- The maximum individual-metabolite AUC in the ranked table must be strictly less than 0.935 (the five-metabolite panel AUC)
- The maximum individual-metabolite AUC must also be less than the Fig 2A OPLS-DA pattern AUC (≥ 0.95)
- The five named metabolites (hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876) must match their AUCs in S2 Fig
- All 195 metabolites from the OPLS-DA model must appear in the ranked output — no dropped rows
- A boolean verification flag for the paper's claim must be present and True only when all benchmark comparisons pass

## Limitations

- Small sample size: the major limitation of the study, difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella carriage
- No independent validation cohort: the resulting metabolite profiles should be validated in an additional, independent cohort, which was not possible in this setting
- All patients presented with gallbladder conditions, so the primary cause for cholecystectomy may impact the metabolite signatures (confounding)
- Lack of controls with other bacterial gallbladder carriage, so metabolites may not be truly Salmonella-specific
- Descriptive nature of the study makes it difficult to reveal what the detected metabolite profiles physiologically reflect

## Evidence

- [supplementary] None of the individual metabolites analyzed had an AUC value as high as or higher than the AUC value of the multivariate metabolite pattern: [evidence span withheld — anchor status: not_found]
- [supplementary] S2 Table lists per-metabolite AUCs with 95% CIs for 195 metabolites: [evidence span withheld — anchor status: not_found]
- [abstract] Five-metabolite panel AUC of 0.935: "Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites"
- [intro] Hypothesis that a combination of co-varying metabolites gives better biomarkers than single metabolites: "we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage"
