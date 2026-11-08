---
name: spearman-correlation-omics-behavioral-factors
description: Use when relating continuous multi-omics factor loadings (e.g., MOFA
  factors from methylation/metabolomics) to behavioral dimension scores in a cohort
  where no family structure needs modeling, and a rank-based association measure is
  preferred.
when_to_use_negative:
- Data come from the NTR twin cohort, where family relatedness must be modeled — use
  GEE with exchangeable working correlation instead of plain Spearman correlation
- The comparison is between discrete SNF clusters rather than continuous factor loadings
  — use Mann–Whitney U (2 clusters) or Kruskal–Wallis (>2 clusters) tests instead
- Input omics data are raw measurements rather than reduced MOFA/MCA factors, and
  no dimension reduction step has been run
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3307
tools:
- name: Nextflow
  role: Workflow manager that orchestrates the association-analysis module running
    the Spearman correlation and FDR correction steps
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containers pulled from Dockerhub provide the runtime environment (R, MOFA2,
    SNFpy dependencies) for the correlation analyses
  repo: https://github.com/Xomics/Docker_containers.git
- name: MOFA2
  role: Produces the omics factors (buccal DNA methylation, urine metabolomics) that
    serve as one side of each Spearman correlation pair
- name: FactoMineR
  role: Computes the MCA behavioral dimension scores that serve as the other side
    of each Spearman correlation pair
- name: SNFpy
  role: Supplies SNF cluster memberships (eigengap-optimal cluster count) that can
    also be correlated with behavioral factors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:7a9d00c1a7bb266dbb1cfb3994222a10a482aeaa397204c59a5c5eeb9a32bf0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - FactoMineR
  - MOFA2
  - Nextflow
  - SNFpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# spearman-correlation-omics-behavioral-factors

## Summary

Determines non-parametric Spearman's rank correlations between MOFA omics factors (derived from buccal DNA methylation and urine metabolomics) and MCA behavioral dimension scores, with FDR correction, as part of the ACTION demonstrator association-testing module.

## When to use

Use when relating continuous multi-omics factor loadings (e.g., MOFA factors from methylation/metabolomics) to behavioral dimension scores in a cohort where no family structure needs modeling, and a rank-based association measure is preferred.

## When NOT to use

- Data come from the NTR twin cohort, where family relatedness must be modeled — use GEE with exchangeable working correlation instead of plain Spearman correlation
- The comparison is between discrete SNF clusters rather than continuous factor loadings — use Mann–Whitney U (2 clusters) or Kruskal–Wallis (>2 clusters) tests instead
- Input omics data are raw measurements rather than reduced MOFA/MCA factors, and no dimension reduction step has been run

## Inputs

- MOFA factor loadings from buccal DNA methylation data
- MOFA factor loadings from urine metabolomics data
- MCA behavioral dimension scores computed with FactoMineR
- SNF cluster memberships (eigengap-optimal cluster count from SNFpy)

## Outputs

- Spearman rank correlation coefficients and p-values per omics factor × behavioral dimension pair
- FDR-corrected p-values appended to association_statistics.csv with cohort, predictor, dimension, test type, and test statistic columns

## How to apply

Load the MOFA factors obtained with MOFA2 from buccal DNA methylation and urine metabolomics data, together with the MCA behavioral dimension scores computed with FactoMineR. For the LUMC-Curium cohort (unrelated individuals), compute Spearman's rank correlation between each omics factor and each behavioral dimension; this complements the GEE models used in the related NTR twin cohort where relatedness must be modeled with an exchangeable working correlation. Apply Spearman's rank correlation to the matrix of all obtained omics and behavioral factors to capture pairwise monotonic associations without distributional assumptions. Correct the resulting correlation p-values for multiple testing across all tested factor–dimension pairs using FDR correction, and record cohort, predictor, behavioral dimension, test type, test statistic, and raw and FDR-corrected p-values per pair in association_statistics.csv.

## Related tools

- **Nextflow** (Workflow manager that orchestrates the association-analysis module running the Spearman correlation and FDR correction steps) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containers pulled from Dockerhub provide the runtime environment (R, MOFA2, SNFpy dependencies) for the correlation analyses) — https://github.com/Xomics/Docker_containers.git
- **MOFA2** (Produces the omics factors (buccal DNA methylation, urine metabolomics) that serve as one side of each Spearman correlation pair)
- **FactoMineR** (Computes the MCA behavioral dimension scores that serve as the other side of each Spearman correlation pair)
- **SNFpy** (Supplies SNF cluster memberships (eigengap-optimal cluster count) that can also be correlated with behavioral factors)

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- association_statistics.csv contains one row per tested omics factor × behavioral dimension combination with test type recorded as Spearman rank correlation
- Every row includes both a raw and an FDR-corrected p-value
- Each row records the cohort (LUMC-Curium for Spearman correlation rows), predictor, and behavioral dimension
- Correlation coefficients are bounded in [-1, 1] and rank-based, robust to non-Gaussian distributions of factor scores
- The workflow completes end-to-end via `nextflow run action.nf` on the synthetic example data without missing association rows

## Limitations

- Spearman correlation is applied to the LUMC-Curium cohort only; the NTR twin cohort requires GEE models with an exchangeable working correlation to adjust for relatedness, so plain correlation would be invalid there
- Privacy regulations prohibit sharing the real omics and clinical data; only a synthetic dataset is provided, limiting direct reuse of the exact association outputs
- Fully FAIR interoperability depends on metadata; describing unsupervised-learning workflow steps required the eNanoMapper Ontology because no matching EDAM term was available
- Factor-based correlations capture monotonic pairwise associations only and do not establish causality between omics factors and externalizing behavior

## Evidence

- [discussion] We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman’s rank correlation: "We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman’s rank correlation"
- [discussion] GEE in NTR vs non-parametric tests in Curium: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [discussion] MOFA factors input: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [readme] Synthetic data example: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [abstract] Workflow orchestration: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
