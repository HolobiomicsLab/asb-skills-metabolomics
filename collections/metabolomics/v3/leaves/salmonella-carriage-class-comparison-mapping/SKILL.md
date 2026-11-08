---
name: salmonella-carriage-class-comparison-mapping
description: Use when reproducing or auditing class-comparison p-values from supervised
  OPLS-DA models of GCxGC-TOFMS plasma metabolite profiles — specifically the five
  comparisons (p = 0.0031, 2.8e-06, 3.1e-05, 3.7e-04, 0.07) reported in Table 1 of
  the carriage study.
when_to_use_negative:
- The analysis goal is unsupervised exploration of group distribution — use PCA scores
  (S1 Fig panel A) rather than OPLS-DA class-comparison significance extraction
- Samples are acute enteric fever cases rather than cholecystectomy patients with
  confirmed gallbladder carriage — the study reports substantial heterogeneity between
  metabolites associated with acute infection and chronic carriage
- A new diagnostic test is being designed from a single cohort — the authors state
  profiles should be validated in an additional independent cohort, which was not
  possible in this setting
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: GCxGC-TOFMS
  role: Source instrument for the 195 plasma metabolites on which the OPLS-DA supervised
    pattern-recognition class comparisons are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:5d7a62b547d2333ccd1382c8132e91d3e430c1ec284bdfd5a27cb8ffa741a555
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# salmonella-carriage-class-comparison-mapping

## Summary

Method for extracting and normalizing the OPLS-DA class-comparison significance statistics (Table 1) from the chronic typhoid carriage metabolomics study, mapping each p-value to its specific model design (two-class combined-carriage, pairwise carriage-vs-control, three-class component separation, and direct carriage-vs-carriage).

## When to use

Use when reproducing or auditing class-comparison p-values from supervised OPLS-DA models of GCxGC-TOFMS plasma metabolite profiles — specifically the five comparisons (p = 0.0031, 2.8e-06, 3.1e-05, 3.7e-04, 0.07) reported in Table 1 of the carriage study.

## When NOT to use

- The analysis goal is unsupervised exploration of group distribution — use PCA scores (S1 Fig panel A) rather than OPLS-DA class-comparison significance extraction
- Samples are acute enteric fever cases rather than cholecystectomy patients with confirmed gallbladder carriage — the study reports substantial heterogeneity between metabolites associated with acute infection and chronic carriage
- A new diagnostic test is being designed from a single cohort — the authors state profiles should be validated in an additional independent cohort, which was not possible in this setting

## Inputs

- PMC-deposited full text of the article (DOI 10.1371/journal.pntd.0006215), including Results, Fig 1, S1 Fig caption, and Table 1
- Verbatim text spans containing p-values, e.g. '(p = 0.0031)', '(p = 2.8*10−6)', '(p = 0.07)'

## Outputs

- oplsda_pvalues.csv with five records and columns: model, comparison, p_value, source_location
- Normalized float p-values (e.g. 2.8e-06) paired with model descriptions and source locations

## How to apply

Parse the PMC-deposited full text of DOI 10.1371/journal.pntd.0006215, focusing on the Results section, Fig 1, and the S1 Fig caption where OPLS-DA statistics are reported for models built on 195 metabolites from GCxGC-TOFMS plasma analysis of Nepali cholecystectomy patients. Extract five values: (1) three-class OPLS-DA separation of S. Typhi carriage from S. Paratyphi A carriage along the second component (p = 0.0031); (2) two-class combined-carriage vs non-carriage controls (p = 2.8*10−6); (3) S. Typhi carriage vs controls (p = 3.1*10−5); (4) S. Paratyphi A carriage vs controls (p = 3.7*10−4); (5) direct S. Typhi vs S. Paratyphi A carriage (p = 0.07). Normalize asterisk scientific notation (e.g. '2.8*10−6' → 2.8e-06) to standard floats. Record each value with its model description, class comparison, and source location (Results paragraph / Fig 1 / Table 1) in oplsda_pvalues.csv with columns model, comparison, p_value, source_location. Validate by cross-checking every extracted p-value against its verbatim text span, e.g. '(p = 2.8*10−6)' and '(p = 0.07)'.

## Related tools

- **GCxGC-TOFMS** (Source instrument for the 195 plasma metabolites on which the OPLS-DA supervised pattern-recognition class comparisons are computed)

## Examples

```
python -c "import pandas as pd; pd.DataFrame([{'model':'two-class OPLS-DA','comparison':'combined carriage vs non-carriage controls','p_value':2.8e-06,'source_location':'Table 1'}]).to_csv('oplsda_pvalues.csv', index=False)"
```

## Evaluation signals

- All five extracted p-values match their verbatim article spans and Table 1: 0.0031, 2.8e-06, 3.1e-05, 3.7e-04, and 0.07
- Asterisk scientific notation is correctly normalized ('2.8*10−6' → 2.8e-06) with no sign or magnitude errors
- oplsda_pvalues.csv contains exactly five rows with the four required columns (model, comparison, p_value, source_location) and no missing fields
- The two-class combined-carriage p-value (2.8e-06) is several orders of magnitude smaller than the direct S. Typhi vs S. Paratyphi A comparison (0.07), consistent with the reported finding that carriage-vs-control separation is significant while carriage-vs-carriage separation is not

## Limitations

- The major limitation of the study was the small sample size, difficult to overcome given ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella
- No independent validation cohort was available, so resulting metabolite profiles could not be externally validated
- All patients presented with gallbladder conditions, so the primary cause of cholecystectomy may confound the metabolite signatures
- No controls with other bacterial gallbladder carriage were included, so Salmonella-specificity of the metabolites is not demonstrated

## Evidence

- [supplementary] the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031): "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [abstract] We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples.: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [supplementary] (A) PCA scores showing the distribution of the three sample groups, S. Typhi carriage, S. Paratyphi A carriage, and non-carriage controls: "(A) PCA scores showing the distribution of the three sample groups, S. Typhi carriage, S. Paratyphi A carriage, and non-carriage controls"
