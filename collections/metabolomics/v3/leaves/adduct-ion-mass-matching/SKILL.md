---
name: adduct-ion-mass-matching
description: Use when you have unlabeled LC/MS feature tables (m/z + retention time
  per feature) and need to estimate what fraction of features correspond to known
  metabolites via negative- or positive-mode adduct matching, e.g. to benchmark preprocessing
  methods.
when_to_use_negative:
- Features already carry metabolite identities (e.g. from standards-based annotation),
  so re-matching adds no information
- The goal is batch effect correction or peak alignment quality itself — this skill
  is a downstream comparison metric, not a correction method
- The data are not small-molecule LC/MS features (e.g. proteomics peptides or genomics
  data), where adduct mass matching is inapplicable
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: xMSAnnotator
  role: 'Performs the metabolite annotation: matches feature m/z values to KEGG compound
    masses via the allowed adduct ion transformations'
- name: apLCMS
  role: Produces the feature tables being compared (single-batch, per-batch, and two-stage
    preprocessing settings)
  repo: https://github.com/tianwei-yu/apLCMS
- name: XCMS
  role: Alternative preprocessing pipeline (centWave peak detection) whose feature
    table is annotated for comparison in Fig. 3b
- name: R
  role: Runtime environment in which xMSAnnotator and apLCMS are executed and proportions
    are computed and plotted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:5a2232bd2d442886a043c11adb79fe26f9b006cfad428c213356f0cb09caf158
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - XCMS
  - apLCMS
  - xMSAnnotator
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# adduct-ion-mass-matching

## Summary

Annotate LC/MS feature tables by matching feature m/z values to known metabolites (e.g. KEGG compounds) through allowed adduct ion transformations such as [M-H]-, using xMSAnnotator. This yields a per-preprocessing-method proportion of features explainable by known metabolites, used to compare feature table quality.

## When to use

Use when you have unlabeled LC/MS feature tables (m/z + retention time per feature) and need to estimate what fraction of features correspond to known metabolites via negative- or positive-mode adduct matching, e.g. to benchmark preprocessing methods.

## When NOT to use

- Features already carry metabolite identities (e.g. from standards-based annotation), so re-matching adds no information
- The goal is batch effect correction or peak alignment quality itself — this skill is a downstream comparison metric, not a correction method
- The data are not small-molecule LC/MS features (e.g. proteomics peptides or genomics data), where adduct mass matching is inapplicable

## Inputs

- Feature table(s) from LC/MS preprocessing (m/z and abundance per feature; e.g. apLCMS cdf.to.ftr output or XCMS centWave features), one per preprocessing setting
- Reference metabolite database with compound masses (KEGG)
- Adduct ion list with exact mass shifts (negative-mode: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-)

## Outputs

- Per-feature annotation results mapping matched features to KEGG compound identities and adduct forms
- Proportion of annotated features per preprocessing setting (e.g. fig3b_annotation_proportions.csv, one proportion per method, each between 0 and 1)
- Comparison bar chart (Fig. 3b-style) of annotation proportions across preprocessing methods

## How to apply

Load each feature table (e.g. ST000868 tables from the three apLCMS preprocessing settings: single-batch, per-batch, and the two-stage hybrid method) and configure xMSAnnotator with KEGG as the reference metabolite database and the paper's negative-mode adduct list: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-. Run annotation separately on each feature table so feature m/z values are matched to KEGG compound masses via the allowed adduct transformations. For each preprocessing setting, compute the proportion of features matched to a KEGG metabolite (annotated features divided by total features). Assemble the proportions into a table (fig3b_annotation_proportions.csv) and draw a bar chart comparing settings. Rationale: because different preprocessing pipelines recover different feature sets, the annotation rate provides a downstream, biology-relevant quality metric — in the source study, XCMS with centWave peak detection yielded a slightly higher matching rate (Fig. 3b), though methods performed similarly.

## Related tools

- **xMSAnnotator** (Performs the metabolite annotation: matches feature m/z values to KEGG compound masses via the allowed adduct ion transformations)
- **apLCMS** (Produces the feature tables being compared (single-batch, per-batch, and two-stage preprocessing settings)) — https://github.com/tianwei-yu/apLCMS
- **XCMS** (Alternative preprocessing pipeline (centWave peak detection) whose feature table is annotated for comparison in Fig. 3b)
- **R** (Runtime environment in which xMSAnnotator and apLCMS are executed and proportions are computed and plotted)

## Examples

```
Rscript -e 'ann <- xMSAnnotator::annotate(ftr_table, adducts=c("[M-H]-","[M-2H]2-","[M-2H+Na]-","[M-2H+K]-","[M-2H+NH4]-","[M-H2O-H]-","[M-H+Cl]2-","[M+Cl]-","[M+2Cl]2-"), db="KEGG"); prop <- sum(ann$match)/nrow(ftr_table)'
```

## Evaluation signals

- fig3b_annotation_proportions.csv exists and contains exactly one proportion per preprocessing setting (single-batch apLCMS, per-batch apLCMS, two-stage apLCMS)
- Each computed proportion lies between 0 and 1 (annotated features / total features)
- The bar chart reproduces the relative ordering reported in Fig. 3b, i.e. similar rates across the three apLCMS settings with XCMS centWave slightly higher
- Each feature is matched only through the specified negative-mode adduct list; no adducts outside the list appear in annotations
- Annotation runs independently per feature table, so counts are not shared or reused across preprocessing settings

## Limitations

- The proportion of KEGG-matched features is a relative comparison metric, not an absolute identification rate — unmatched features may be real metabolites absent from KEGG or in unlisted adduct forms
- Matching relies only on exact m/z via adduct transformations (no retention time or MS/MS evidence), so annotations are putative and can be ambiguous among isomeric compounds
- Treating all samples as a single batch during preprocessing can cause peak alignment errors that cannot be corrected post hoc, which would distort the downstream annotation proportions themselves
- Results are database-dependent: swapping KEGG for another reference database changes the matched proportion and is not directly comparable

## Evidence

- [other] the methods performed similarly, with XCMS with centWave peak detection yielding a slightly higher rate of matching (Fig. 3b): [evidence span withheld — anchor status: not_found]
- [other] negative-mode adduct list: [evidence span withheld — anchor status: not_found]
- [abstract] batch-aware preprocessing better downstream results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] workflow list of LC/MS processing steps: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
- [methods] apLCMS raw data processing entry point: [evidence span withheld — anchor status: not_found]
