---
name: ruv-unwanted-variation-estimation
description: Use when a metabolomics featuredata matrix is subject to unwanted variation
  (batch, matrix effects) and negative-control/QC metabolites are available so ruv2,
  ruvrand, or ruvrandclust can be fitted with a known factor model.
when_to_use_negative:
- When no QC metabolites or negative controls are designated in metabolitedata — the
  NormQcmets approaches (including the ruv methods) rely on internal/external standards
  and other quality control metabolites.
- When normalization should instead be based on QC samples — use NormQcsamples (QC-RLSC
  signal correction) rather than NormQcmets.
- When a simple scaling normalization suffices — NormScaling (median, mean, sum, ref)
  is the lower-complexity alternative.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: NormalizeMets
  role: R package providing NormQcmets, which dispatches the ruv2, ruvrand, and ruvrandclust
    unwanted-variation methods
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: R
  role: runtime environment (version 3.4.3 or higher) in which NormalizeMets functions
    are executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:034ea9078d2bf0e9c04d29f326a43b78d72d35d12173c61b9c610024e1a00204
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ruv-unwanted-variation-estimation

## Summary

Estimates and removes unwanted variation in metabolomics feature data using the remove-unwanted-variation (RUV) family of methods — ruv2, ruvrand, and ruvrandclust — dispatched through the NormalizeMets NormQcmets function, which also offers the is, nomis, and ccmn internal-standard alternatives.

## When to use

Use when a metabolomics featuredata matrix is subject to unwanted variation (batch, matrix effects) and negative-control/QC metabolites are available so ruv2, ruvrand, or ruvrandclust can be fitted with a known factor model.

## When NOT to use

- When no QC metabolites or negative controls are designated in metabolitedata — the NormQcmets approaches (including the ruv methods) rely on internal/external standards and other quality control metabolites.
- When normalization should instead be based on QC samples — use NormQcsamples (QC-RLSC signal correction) rather than NormQcmets.
- When a simple scaling normalization suffices — NormScaling (median, mean, sum, ref) is the lower-complexity alternative.

## Inputs

- featuredata matrix (samples x metabolites, unique sample names as row names, unique metabolite names as columns)
- sampledata dataframe (sample-specific factors such as gender, Age, bmi)
- metabolitedata dataframe designating metabolites as internal/external standards or controls (IS==1 flags QC metabolites)
- method argument string: 'ruv2', 'ruvrand', or 'ruvrandclust'
- factormat design matrix of factors of interest (e.g. model.matrix(~gender+Age+bmi, sampledata))
- k, the number of unwanted-variation factors to estimate (e.g. k=2)

## Outputs

- normalized featuredata matrix with the same samples-by-metabolites dimensions as the input
- estimated unwanted-variation factors for the ruv methods
- method-specific extras such as factors/ncomp and lambda

## How to apply

Prepare featuredata, sampledata, and metabolitedata, then log-transform with LogTransform (zerotona=TRUE) and impute missing values via MissingValues (knn, k=10, feature.cutoff=0.8, sample.cutoff=0.8). Call NormQcmets with method='ruv2', 'ruvrand', or 'ruvrandclust'; NormQcmets internally matches the method argument against c('is','nomis','ccmn','ruv2','ruvrand','ruvrandclust'). For ruv2 supply factormat=model.matrix(~gender+Age+bmi, sampledata) and k=2 (the number of unwanted-variation factors); qcmets are identified as which(metabolitedata$IS==1). The function returns normalized featuredata of the same samples-by-metabolites dimensions plus method-specific extras, including the estimated unwanted-variation factors for the ruv methods.

## Related tools

- **NormalizeMets** (R package providing NormQcmets, which dispatches the ruv2, ruvrand, and ruvrandclust unwanted-variation methods) — https://github.com/metabolomicstats/NormalizeMets
- **R** (runtime environment (version 3.4.3 or higher) in which NormalizeMets functions are executed) — https://cran.r-project.org/

## Examples

```
NormQcmets(featuredata=imp$featuredata, factormat=model.matrix(~gender+Age+bmi, sampledata), method="ruv2", qcmets=which(metabolitedata$IS==1), k=2)
```

## Evaluation signals

- The dispatch accepts exactly the six documented method strings c('is','nomis','ccmn','ruv2','ruvrand','ruvrandclust') and rejects others.
- Each route returns a normalized data matrix with the same samples-by-metabolites dimensions as the input featuredata.
- For ruv2, factormat is built from the factors of interest (e.g. ~gender+Age+bmi) and k is set explicitly (k=2).
- QC metabolites used by the normalization are exactly those flagged which(metabolitedata$IS==1).
- The returned object includes the estimated unwanted-variation factors for the ruv methods.

## Limitations

- RUV-based methods require the metabolitedata dataframe to designate QC/negative-control metabolites; without IS or control annotations the route cannot be applied.
- Unwanted variation in metabolomics data arises from batch effects, matrix effects, and confounding biological variation, so the factor model (factormat) and k must be chosen per study.
- Missing values must be handled first (knn imputation with feature/sample cutoffs of 0.8), otherwise the normalization operates on incomplete data.

## Evidence

- [readme] Metabolomics data are inevitably subject to a component of unwanted variation, due to factors such as batch effects, matrix effects, and confounding biological variation.: [evidence span withheld — anchor status: external_doc]
- [readme] metabolitedata contains metabolite-specific information in a separate dataframe. These information can include, but is not limited to, designation of metabolites as internal/external standards: [evidence span withheld — anchor status: external_doc]
