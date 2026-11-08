---
name: annotation-power-and-tpr-metric-computation
description: Use when comparing metabolite annotations from two annotation methods
  (e.g., ML vs rule-based) at FDR < 10% against an independent MS/MS-validated LC-MS/MS
  reference set, to compute MAP and TPR metrics.
when_to_use_negative:
- The imaging annotations have not been filtered at FDR < 10%, since the metric reproduction
  assumes FDR-controlled annotation lists
- No independent MS/MS-validated LC-MS/MS ground-truth identifications are available,
  making TPR computation impossible
- Comparing annotation quality on a single dataset rather than across the multi-dataset
  (10 mouse brain) context where median MAP is defined
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: Machine-learning annotator producing the FDR-controlled formula+adduct annotations
    (one arm of the TPR/MAP comparison)
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform whose rule-based MSM scoring provides the comparison arm and whose
    python-client/Jupyter tooling supports annotation data analysis
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:d09207c9c3ab06478967c3e851b07772deaa5e8656529554b66b4688eb6c920d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# annotation-power-and-tpr-metric-computation

## Summary

Benchmarking procedure from the METASPACE-ML paper that validates imaging-MS metabolite annotations (formula+adduct pairs) against MS/MS-validated bulk LC-MS/MS identifications and reports median METASPACE annotation power (MAP) and true positive rate (TPR) for the ML and rule-based MSM approaches.

## When to use

Use when comparing metabolite annotations from two annotation methods (e.g., ML vs rule-based) at FDR < 10% against an independent MS/MS-validated LC-MS/MS reference set, to compute MAP and TPR metrics.

## When NOT to use

- The imaging annotations have not been filtered at FDR < 10%, since the metric reproduction assumes FDR-controlled annotation lists
- No independent MS/MS-validated LC-MS/MS ground-truth identifications are available, making TPR computation impossible
- Comparing annotation quality on a single dataset rather than across the multi-dataset (10 mouse brain) context where median MAP is defined

## Inputs

- Supplementary Data 8: formulas+adducts annotated at FDR < 10% by METASPACE-ML and MSM for 10 mouse brain MALDI-FTICR imaging datasets
- Supplementary Data 9: MS/MS-validated bulk LC-MS/MS metabolite identifications (reference ground truth)

## Outputs

- validation_metrics.csv: metric table with per-approach median MAP (METASPACE-ML vs MSM), overall TPR, and exclusive-ion TPR (72% vs 11%)

## How to apply

Load Supplementary Data 8, which contains formulas+adducts annotated at FDR < 10% by METASPACE-ML and MSM across the 10 mouse brain MALDI-FTICR imaging datasets, and Supplementary Data 9 with the MS/MS-validated bulk LC-MS/MS identifications serving as ground truth. Using pandas, perform set-based intersection of formula+adduct identifier pairs between each approach's imaging annotations and the LC-MS/MS reference to label annotations as MS/MS-validated true positives. Compute per-approach median METASPACE annotation power (MAP) across the 10 datasets, then compute the overall TPR of validated annotations per approach, and separately restrict to ions annotated exclusively by one approach to expose complementarity. A correct run reproduces median MAP 0.57 (METASPACE-ML) vs 0.43 (MSM), overall TPR 91%, and exclusive-ion TPR 72% vs 11%; write the metric table to validation_metrics.csv.

## Related tools

- **METASPACE-ML** (Machine-learning annotator producing the FDR-controlled formula+adduct annotations (one arm of the TPR/MAP comparison)) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform whose rule-based MSM scoring provides the comparison arm and whose python-client/Jupyter tooling supports annotation data analysis) — https://github.com/metaspace2020/metaspace

## Examples

```
import pandas as pd; ml = pd.read_csv('SD8_ml.csv'); truth = pd.read_csv('SD9_lcmsms.csv'); tp = ml.merge(truth, on=['formula','adduct']); pd.DataFrame({'tpr': [len(tp)/len(ml)]}).to_csv('validation_metrics.csv')
```

## Evaluation signals

- Overall TPR across both approaches' validated annotations reaches approximately 91%
- Median MAP across the 10 datasets is approximately 0.57 for METASPACE-ML and 0.43 for MSM
- Exclusive-ion TPR is approximately 72% for METASPACE-ML vs 11% for the rule-based MSM approach
- Every matched annotation is compared as a formula+adduct pair via set intersection, so no partial/formula-only matches are counted as true positives
- Output file validation_metrics.csv contains one row per approach per metric category (overall and exclusive-ion)

## Limitations

- Some annotations are captured by MSM but not METASPACE-ML due to MSM's sensitivity to its feature scores, particularly the rho_spatial score, so neither approach dominates exclusively
- The validation is specific to the MALDI-FTICR mouse brain context; generalization to underrepresented data types (e.g., QTOF, single-cell metabolomics) is not established

## Evidence

- [abstract] ML outperforms MSM benchmark: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
- [intro] MSM rule-based limitation: "a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations"
- [discussion] MSM-exclusive annotations: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [intro] FDR-controlled ML annotation: "we introduce METASPACE-ML, a machine-learning model for FDR-controlled metabolite annotation within specific contexts in imaging MS"
- [readme] Analysis tooling: [evidence span withheld — anchor status: external_doc]
