---
name: creatinine-based-urine-normalization
description: Use when preprocessing urine metabolite intensity tables (e.g., Synthetic_data/synthetic_metabolomics.csv)
  where between-sample dilution differences confound comparison, and intensities must
  be corrected to creatinine levels prior to Pareto scaling.
when_to_use_negative:
- Data are already concentration-corrected or derived from non-urine matrices (e.g.,
  buccal DNA methylation betas) where creatinine adjustment does not apply
- The analysis goal is a different modality step, such as selecting the top 10% most
  variable methylation probes or MOFA factorization — normalization applies only to
  the urine metabolomics block
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Nextflow
  role: Workflow manager executing the metabolomics preprocessing process (action.nf)
    that performs the creatinine normalization and Pareto scaling
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime pulling software dependencies for the preprocessing steps
    from Dockerhub via nextflow.config
  repo: https://github.com/Xomics/Docker_containers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:807d512bbc8c699d12b0fa7a105e8d5912764a8e3c0bb2165d1da5f9044a9d67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# creatinine-based-urine-normalization

## Summary

Sample-wise normalization of urine metabolomics intensity data by adjusting each metabolite to the sample's creatinine level, correcting for inter-individual differences in urine concentration before downstream scaling and analysis in the ACTIONdemonstrator multi-omics workflow.

## When to use

Use when preprocessing urine metabolite intensity tables (e.g., Synthetic_data/synthetic_metabolomics.csv) where between-sample dilution differences confound comparison, and intensities must be corrected to creatinine levels prior to Pareto scaling.

## When NOT to use

- Data are already concentration-corrected or derived from non-urine matrices (e.g., buccal DNA methylation betas) where creatinine adjustment does not apply
- The analysis goal is a different modality step, such as selecting the top 10% most variable methylation probes or MOFA factorization — normalization applies only to the urine metabolomics block

## Inputs

- Urine metabolomics intensity table (e.g., Synthetic_data/synthetic_metabolomics.csv)
- Per-sample creatinine levels (as used by the Nextflow process in action.nf)
- Sample-level QC metadata (urine-quality/protocol criteria, e.g., MtblmcsValues/Variables from XOmics_NTR_ACTION_MtblmcsValues.tsv)

## Outputs

- QC-filtered, creatinine-normalized, Pareto-scaled metabolomics matrix (preprocessed_metabolomics.csv in the workflow output directory)

## How to apply

Load the metabolomics intensity table (Synthetic_data/synthetic_metabolomics.csv) with pandas or R, mirroring the input expected by the Nextflow process in action.nf of the ACTIONdemonstrator_workflow. First apply sample-level QC exclusions (e.g., samples failing urine-quality/protocol criteria such as menstruating subjects, >2 h delay before freezer storage, positive nitrites, or protein >0.3) and discard samples and metabolites with 15% or more missing values. Then perform sample-wise normalization by dividing/adjusting each metabolite intensity to the sample's creatinine level, which corrects for urine dilution as described in the article. Follow with metabolite-wise Pareto scaling so each metabolite is mean-centered with variance adjusted by the square root of its standard deviation. The rationale is that urine concentration varies substantially between subjects and collections, so creatinine adjustment is required before valid cross-sample and cross-omics (MOFA) comparisons.

## Related tools

- **Nextflow** (Workflow manager executing the metabolomics preprocessing process (action.nf) that performs the creatinine normalization and Pareto scaling) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime pulling software dependencies for the preprocessing steps from Dockerhub via nextflow.config) — https://github.com/Xomics/Docker_containers

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv
```

## Evaluation signals

- Output table preprocessed_metabolomics.csv exists in the workflow output directory
- Metabolite identifiers are retained as columns/rows after normalization and scaling
- No retained sample or metabolite exceeds the 15% missingness threshold
- Scaled metabolite columns have mean approximately zero with variance adjusted by the square root of the standard deviation (Pareto scaling)
- Per-sample intensity distributions are consistent with creatinine-adjusted concentrations (dilution-driven intensity differences reduced)

## Limitations

- Only a synthetic dataset is shared because privacy regulations prohibit sharing the real omics measurements, so normalization behavior on real ACTION Biomarker Study data must be inferred from the workflow
- Normalization depends on prior QC exclusions (menstruation, >2 h storage delay, protocol violations, leukocyte/nitrite/protein/glucose/blood criteria, illness reports); violations of these upstream filters propagate into the normalized matrix
- Full FAIR reuse requires the containerized environment; running outside Nextflow/Docker may not reproduce the exact preprocessing behavior

## Evidence

- [discussion] Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels: "Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels"
- [discussion] Samples and metabolites with 15% or more of missing values were discarded.: "Samples and metabolites with 15% or more of missing values were discarded."
- [other] Perform sample-wise normalization by adjusting metabolite intensities to each sample's creatinine level to correct for urine dilution.: [evidence span withheld — anchor status: not_found]
- [abstract] The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies.: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
- [readme] nextflow run action.nf --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv: "nextflow run action.nf 
            --output dir/of/choice
            --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv"
