---
name: metabolomics-preprocessing-and-covariate-residualization
description: Use when raw metabolite intensity data (e.g., Metabolon platform output
  from Metabolomics Workbench) must be cleaned, batch/run-day normalized, imputed,
  and converted to covariate-adjusted partial residuals before kernel machine or pathway
  association tests.
when_to_use_negative:
- Input is already a normalized, imputed, covariate-adjusted residual matrix — reapplying
  would double-filter and distort z-score outlier removal
- The downstream method requires raw (untransformed) intensities rather than log-transformed,
  standardized partial residuals
- Missingness exceeds what kNN imputation (k = 10) can handle, e.g., metabolites missing
  in >20% of samples, which the pipeline excludes rather than imputes
edam_operation: http://edamontology.org/operation_2943
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Language in which the full preprocessing and residualization workflow (normalization,
    imputation, regression) is implemented
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: impute (R package)
  role: Performs k-nearest neighbors imputation (k = 10) of missing metabolite values
    across metabolites
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:bbdb1c9984ac9e62370c224ae7f3c587c8b4c9a0d5c75d1dc3a49a49ad30aed1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  repo_url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  - impute (R package)
  edam_operation: http://edamontology.org/operation_2943
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-preprocessing-and-covariate-residualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw metabolomics intensities (run-day median normalization, KNN imputation, log transform, standardization, outlier/missingness filtering) and residualize each metabolite against covariates via linear regression, producing partial residuals for downstream pathway association testing.

## When to use

Use when raw metabolite intensity data (e.g., Metabolon platform output from Metabolomics Workbench) must be cleaned, batch/run-day normalized, imputed, and converted to covariate-adjusted partial residuals before kernel machine or pathway association tests.

## When NOT to use

- Input is already a normalized, imputed, covariate-adjusted residual matrix — reapplying would double-filter and distort z-score outlier removal
- The downstream method requires raw (untransformed) intensities rather than log-transformed, standardized partial residuals
- Missingness exceeds what kNN imputation (k = 10) can handle, e.g., metabolites missing in >20% of samples, which the pipeline excludes rather than imputes

## Inputs

- Raw metabolite intensity matrix from Metabolon Global Metabolomics platform (e.g., P100 plasma, Metabolomics Workbench PR000907)
- Run-day/batch annotation per sample
- Subject covariates: white blood cell count, percent eosinophil, percent lymphocytes, percent monocytes, percent neutrophils, hemoglobin
- Phenotype complete-case design matrix (e.g., log FEV1/FVC, percent emphysema)

## Outputs

- Normalized, imputed, log-transformed and standardized metabolite matrix
- Filtered metabolite table (metabolites missing >20% of samples excluded)
- Per-metabolite partial residuals from covariate-adjusted linear regressions
- Complete-case subject sets (1,113 FEV1/FVC and 1,065 percent emphysema in the COPDGene application)

## How to apply

Divide each metabolite intensity by its run-day median and multiply by the overall metabolite median to correct run-day batch effects. Impute missing values with k-nearest neighbors (k = 10) using the R package impute, then natural log transform and standardize the data. Remove subjects whose aggregate metabolite median z-score exceeds 3.5 SD from the mean (n = 6 in COPDGene) and exclude metabolites missing in more than 20% of samples. For each metabolite, fit a linear regression controlling for white blood cell count, percent eosinophil, percent lymphocytes, percent monotes, percent neutrophils, and hemoglobin, and use the partial residuals as the observed metabolomics data for downstream testing; in the COPDGene application this left 1,113 complete cases for FEV1/FVC and 1,065 for percent emphysema.

## Related tools

- **R** (Language in which the full preprocessing and residualization workflow (normalization, imputation, regression) is implemented) — https://github.com/CharlieCarpenter/PaIRKAT
- **impute (R package)** (Performs k-nearest neighbors imputation (k = 10) of missing metabolite values across metabolites)

## Examples

```
Rscript -e 'library(impute); dat <- impute.knn(raw_intensities, k=10)$data; dat <- log(t(t(dat)/runDayMedian*overallMedian)); res <- residuals(lm(dat ~ covariates))'
```

## Evaluation signals

- Subject-level outlier filter removes subjects with aggregate metabolite median z-scores > 3.5 SD (n = 6 in COPDGene)
- Metabolite retention check: all retained metabolites have missingness ≤ 20% of samples
- Complete-case counts match expectations (1,113 for FEV1/FVC; 1,065 for percent emphysema)
- Residualized metabolite values are orthogonal to the six adjustment covariates (white blood cell count, eosinophils, lymphocytes, monocytes, neutrophils, hemoglobin) by construction of the per-metabolite linear regressions
- Downstream association testing (e.g., 28 KEGG pathways with ≥10 measured metabolites) proceeds without further imputation errors

## Limitations

- Filtered dataset did not contain every metabolite within the 28 KEGG pathways selected, so some analyzed pathways had fewer than 10 measured metabolites
- Outlier rule (aggregate median z-score > 3.5 SD) is cohort-specific; thresholds may not transfer to other studies
- Downstream method performance differences only manifest on downsampled subsets (100–500 subjects); with full cohorts >1,000 observations, method p-values were indistinguishable
- Software/tool versions for R and impute were not reported

## Evidence
