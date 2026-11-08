---
name: behavioral-data-matrix-preprocessing
description: Use when the behavioral input to the workflow (ASEBA CBCL item responses
  supplied via --behavioral_data) contains missing item entries that must be imputed
  before behavioral factors can be derived with MCA.
when_to_use_negative:
- The CBCL item matrix has no missing values, so imputation would be unnecessary
- The missing data belong to urine metabolomics samples/metabolites — those follow
  the article's discard rule (samples and metabolites with 15% or more missing values
  were discarded) rather than missForest imputation
- The goal is DNA methylation or metabolomics factor extraction (MOFA2/SNF steps),
  which does not consume the behavioral item matrix
tools:
- name: missForest
  role: R library (v1.4) performing the iterative nonparametric random-forest imputation
    of missing CBCL items
- name: Nextflow
  role: Workflow manager running the behavioral imputation step as part of the modular
    action.nf pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container capturing the software dependencies (R with missForest) so the imputation
    step runs reproducibly
  repo: https://github.com/Xomics/Docker_containers.git
- name: FactoMineR
  role: 'Consumer of the imputed matrix: applied MCA to obtain factors for the behavioral
    data'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:e4ff7c6dfcc3828237ffd524c8eda90dfa6593647f3f848837d3eca3ee65449d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - FactoMineR
  - Nextflow
  - missForest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# behavioral-data-matrix-preprocessing

## Summary

Prepares the parent-rated ASEBA CBCL externalizing item matrix for factor analysis in the ACTION demonstrator multi-omics workflow by imputing missing item responses with the non-parametric random-forest method missForest (R library v1.4). The completed behavioral matrix is required so downstream MCA (R library FactoMineR 2.4) can extract behavioral factors without missing-value errors.

## When to use

Use when the behavioral input to the workflow (ASEBA CBCL item responses supplied via --behavioral_data) contains missing item entries that must be imputed before behavioral factors can be derived with MCA.

## When NOT to use

- The CBCL item matrix has no missing values, so imputation would be unnecessary
- The missing data belong to urine metabolomics samples/metabolites — those follow the article's discard rule (samples and metabolites with 15% or more missing values were discarded) rather than missForest imputation
- The goal is DNA methylation or metabolomics factor extraction (MOFA2/SNF steps), which does not consume the behavioral item matrix

## Inputs

- ASEBA CBCL item-response matrix (CSV, e.g. Synthetic_data/synthetic_cbcl_data.csv) with subject identifiers as row labels, supplied via --behavioral_data
- Nextflow process context of action.nf (Docker/Singularity container with R and missForest 1.4)

## Outputs

- Complete behavioral item matrix (imputed_cbcl_data.csv) with no remaining missing values, original item column order and subject row order preserved
- Downstream input for multiple correspondence analysis (MCA, R library FactoMineR 2.4)

## How to apply

Load the behavioral data file passed via the --behavioral_data input (in the packaged example run: Synthetic_data/synthetic_cbcl_data.csv), keeping subject identifiers as row labels and preserving the original CBCL item columns. Identify the missing-value pattern, treating unobserved items as entries to be imputed. Run the nonparametric random-forest imputation with the R library missForest (v1.4), which iteratively fills in each missing item by regressing it on the other items until convergence. Assemble the completed matrix preserving original row and column order and write it as imputed_cbcl_data.csv. Validate that the output has no remaining missing values and the same subject and item sets as the input before it is passed to the MCA step (FactoMineR 2.4) of the Nextflow workflow.

## Related tools

- **missForest** (R library (v1.4) performing the iterative nonparametric random-forest imputation of missing CBCL items)
- **Nextflow** (Workflow manager running the behavioral imputation step as part of the modular action.nf pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container capturing the software dependencies (R with missForest) so the imputation step runs reproducibly) — https://github.com/Xomics/Docker_containers.git
- **FactoMineR** (Consumer of the imputed matrix: applied MCA to obtain factors for the behavioral data)

## Examples

```
nextflow run action.nf --output dir/of/choice --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- Output file imputed_cbcl_data.csv exists after the workflow step completes
- Zero remaining missing values in the imputed CBCL matrix
- The imputed matrix contains exactly the same set of subjects and CBCL items as the input matrix
- Original row (subject) order and column (item) order are preserved relative to synthetic_cbcl_data.csv
- The completed matrix loads into the downstream MCA step (FactoMineR 2.4) without missing-value or schema errors

## Limitations

- Privacy regulations prohibit sharing the real, potentially personally identifiable behavioral/clinical data, so only the synthetic CBCL dataset in Synthetic_data/ demonstrates the workflow
- missForest convergence depends on iterative random-forest regression on the remaining items; highly sparse item patterns may affect imputed values
- This skill covers only the behavioral item matrix; missingness in omics data is handled by different rules (e.g., 15% missing-value discard threshold for metabolomics)

## Evidence

- [methods] Run the nonparametric random-forest imputation with the R library missForest (v1.4), which iteratively fills in each missing item by regressing it on the other items: [evidence span withheld — anchor status: not_found]
- [methods] verify that the output imputed_cbcl_data.csv exists, has no remaining missing values, and contains the same set of subjects and CBCL items as the input matrix: [evidence span withheld — anchor status: not_found]
- [discussion] applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [readme] Synthetic data is provided as example data in `Synthetic_data/`: "Synthetic data is provided as example data in `Synthetic_data/`"
