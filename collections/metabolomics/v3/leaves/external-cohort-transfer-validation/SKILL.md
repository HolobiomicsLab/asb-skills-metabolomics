---
name: external-cohort-transfer-validation
description: Use when a cross-validated metabolite-prediction model exists and an
  independent paired microbiome-metabolome cohort is available to test transfer without
  retraining.
when_to_use_negative:
- No independent external cohort with paired microbiome-metabolome samples exists
  — cross-validation alone cannot substitute for external transfer validation.
- The external metabolome features are unannotated and the evaluation goal requires
  annotated-metabolite reporting, since IBD annotated SCC (0.309) relies on the annotation
  file.
- The goal is internal performance estimation (e.g. model comparison via 10-iteration
  10-fold cross-validation) rather than transfer to a new cohort.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3697
tools:
- name: MiMeNet
  role: Neural network (MLPNN) framework whose MiMeNet_train.py script applies the
    trained model to the external cohort via -external_micro/-external_metab and computes
    external-validation SCCs
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:4924a1e23721b30df42e5d249a2e576e20d19ce9e2f611b6f244075169231ec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# external-cohort-transfer-validation

## Summary

Apply a trained MiMeNet model to an independent external cohort (the IBD external test set) to obtain external-validation Spearman correlation coefficients (SCCs), testing that cross-validated performance transfers beyond the training cohort.

## When to use

Use when a cross-validated metabolite-prediction model exists and an independent paired microbiome-metabolome cohort is available to test transfer without retraining.

## When NOT to use

- No independent external cohort with paired microbiome-metabolome samples exists — cross-validation alone cannot substitute for external transfer validation.
- The external metabolome features are unannotated and the evaluation goal requires annotated-metabolite reporting, since IBD annotated SCC (0.309) relies on the annotation file.
- The goal is internal performance estimation (e.g. model comparison via 10-iteration 10-fold cross-validation) rather than transfer to a new cohort.

## Inputs

- Trained MiMeNet model and network_parameters.txt (JSON file of hyperparameters: number of layers, layer size, L2 penalty, dropout rate)
- External test set microbiome count CSV (samples by microbial features, e.g. data/IBD/microbiome_external.csv)
- External test set metabolome count CSV (e.g. data/IBD/metabolome_external.csv)
- Metabolite annotation CSV (data/IBD/metabolome_annotation.csv) for restricting IBD evaluation to annotated metabolites
- Shuffled-sample background SCC distribution for well-predicted metabolite flagging

## Outputs

- External-validation Spearman correlation coefficients (SCC) per metabolite (target mean ~0.275 for IBD External)
- Set of well-predicted metabolites whose SCC exceeds the 95th percentile of the shuffled background
- Per-dataset summary table (e.g. mimenet_scc_summary.csv) with mean SCC and well-predicted metabolite counts

## How to apply

Train MiMeNet on the paired IBD (PRISM) microbiome/metabolome CSVs via MiMeNet_train.py, then pass the external cohort as -external_micro (microbiome_external.csv) and -external_metab (metabolome_external.csv) so the trained model is applied to the independent samples. Evaluate performance with per-metabolite Spearman correlation coefficients (SCC), restricting IBD reporting to annotated metabolites supplied via the -annotation file (metabolome_annotation.csv). Expected external validation SCC is 0.275 (IBD External) with well-predicted metabolites identified by comparing each SCC to the 95th percentile of a shuffled-sample background distribution. Transfer is successful if the external SCC falls within tolerance of the reported value and well-predicted counts are of the same order as reported (366 IBD annotated, 143 CF, 29 soil).

## Related tools

- **MiMeNet** (Neural network (MLPNN) framework whose MiMeNet_train.py script applies the trained model to the external cohort via -external_micro/-external_metab and computes external-validation SCCs) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -external_micro data/IBD/microbiome_external.csv -external_metab data/IBD/metabolome_external.csv -micro_norm None -metab_norm CLR -net_params results/IBD/network_parameters.txt -annotation data/IBD/metabolome_annotation.csv -num_run_cv 10 -output IBD
```

## Evaluation signals

- External-validation mean SCC for the IBD external set falls within tolerance of the reported 0.275 (IBD External).
- Well-predicted metabolite counts (SCC above the 95th percentile of the shuffled background) are of the same order as reported: 366 (IBD annotated), 143 (CF), 29 (soil).
- mimenet_scc_summary.csv exists and contains a per-dataset table with mean SCC and well-predicted counts.
- The external run used the same network_parameters.txt hyperparameters (layers, layer size, L2 penalty, dropout) as the cross-validated training run.

## Limitations

- Not all metabolites may be associated with microbes, so some metabolites will have lower prediction correlations, resulting in an overall lower mean correlation across all metabolites.
- MiMeNet is still limited in providing biological plausibility and mechanistic insights despite predictive accuracy.
- A higher threshold value was observed for the soil data, possibly due to longitudinal observations (unresolved in the paper).

## Evidence

- [other] For the IBD dataset, apply the trained model to the external test set using the external_micro and external_metab inputs to obtain external-validation SCCs.: [evidence span withheld — anchor status: not_found]
- [readme] README external test set parameters: "external_micro    Comma delimited file representing matrix of samples by microbial features"
- [other] Workflow step 7 with tolerance values: [evidence span withheld — anchor status: not_found]
- [methods] Well-predicted metabolite definition: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [discussion] External validation claim: "we showed that MiMeNet models can produce a robust and accurate prediction of the community metabolome based on the microbial taxa abundance in both cross-validation and external validation."
