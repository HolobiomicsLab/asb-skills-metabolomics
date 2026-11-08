---
name: urine-metabolomics-qc-sample-exclusion
description: Use when preprocessing urine metabolomics intensity tables (e.g. Mtblmcs
  values/variables from the ACTION Biomarker Study) and samples must be filtered on
  collection-protocol, dipstick, and health criteria prior to creatinine normalization.
when_to_use_negative:
- Input is epigenomics (DNA methylation) data, where instead only the top 10% most
  variable autosomal methylation probes surviving QC are retained
- The metabolomics data are not urine-based (no creatinine/dipstick or urine-collection
  metadata available to evaluate the exclusion criteria)
- The data are synthetic example files (Synthetic_data/synthetic_metabolomics.csv)
  used to demonstrate the workflow rather than real cohort samples subject to these
  clinical exclusions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Nextflow
  role: Workflow manager implementing the data-preparation pipeline (action_file_preparation.nf)
    that applies the sample-exclusion and missingness filters
  repo: https://github.com/Xomics/ACTIONdemonstrator_preprocessing
- name: Docker
  role: Containerizes the software dependencies of the QC/preprocessing processes;
    containers pulled from Dockerhub or built locally via Xomics/Docker_containers
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:34605255a4b494ca1c2c28f8378f45269fc2657b138c1e29abfae3077d4fc936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_preprocessing
  tools:
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# urine-metabolomics-qc-sample-exclusion

## Summary

Applies the ACTIONdemonstrator cohort-specific sample-level QC rules to urine metabolomics data: excludes samples violating urine-quality and sampling-protocol criteria, then discards samples (and metabolites) with 15% or more missing values before downstream normalization.

## When to use

Use when preprocessing urine metabolomics intensity tables (e.g. Mtblmcs values/variables from the ACTION Biomarker Study) and samples must be filtered on collection-protocol, dipstick, and health criteria prior to creatinine normalization.

## When NOT to use

- Input is epigenomics (DNA methylation) data, where instead only the top 10% most variable autosomal methylation probes surviving QC are retained
- The metabolomics data are not urine-based (no creatinine/dipstick or urine-collection metadata available to evaluate the exclusion criteria)
- The data are synthetic example files (Synthetic_data/synthetic_metabolomics.csv) used to demonstrate the workflow rather than real cohort samples subject to these clinical exclusions

## Inputs

- Urine metabolomics values table (e.g. XOmics_NTR_ACTION_MtblmcsValues.tsv)
- Urine metabolomics variables table (e.g. XOmics_NTR_ACTION_MtblmcsVariables.tsv)
- ACTION metabolomics/biomarkers dictionary (ACTION_dictionary.xlsx)
- Phenotype/health-status and urine dipstick measurements for the sample-level exclusion criteria

## Outputs

- QC-filtered urine metabolomics matrix containing only samples passing the exclusion criteria and retaining only samples/metabolites below the 15% missingness threshold, ready for creatinine-based normalization

## How to apply

Load the urine metabolomics values/variables tables (e.g. XOmics_NTR_ACTION_MtblmcsValues.tsv with its variables file and ACTION_dictionary.xlsx) into the Nextflow data-preparation pipeline (action_file_preparation.nf in Xomics/ACTIONdemonstrator_preprocessing), which runs in Docker/Singularity containers. Exclude samples from subjects who have started menstruating, subjects in whom the time between urine sample collection and freezer storage exceeded 2 hours, subjects with severe sampling protocol violations (e.g., not putting a lid on the container), and subjects with abnormal urine dipstick results: leukocyte count above trace, nitrites 'positive high', protein level >0.3, glucose above trace, or blood above trace. Also exclude samples from subjects reporting flu, inflammation, vomiting, abdominal pain, or general health problems, since these confound the urine metabolome. After these cohort-specific exclusions, discard samples and metabolites with 15% or more missing values. The rationale is that urine dilution, contamination, and acute health states distort metabolite intensities, so these samples must be removed before sample-wise creatinine normalization and Pareto scaling in the downstream action.nf workflow.

## Related tools

- **Nextflow** (Workflow manager implementing the data-preparation pipeline (action_file_preparation.nf) that applies the sample-exclusion and missingness filters) — https://github.com/Xomics/ACTIONdemonstrator_preprocessing
- **Docker** (Containerizes the software dependencies of the QC/preprocessing processes; containers pulled from Dockerhub or built locally via Xomics/Docker_containers) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action_file_preparation.nf --output /dir/of/choice --mtblmcs_values /mnt/workspace/Data/metabolomics_and_biomarkers/upload/XOmics_NTR_ACTION_MtblmcsValues.tsv --mtblmcs_variables XOmics_NTR_ACTION_MtblmcsVariables.tsv --mtblmcs_dictionary ACTION_dictionary.xlsx
```

## Evaluation signals

- No retained sample belongs to a subject meeting any exclusion criterion (menstruation, >2 h collection-to-freezer time, severe protocol violation, abnormal dipstick, or reported flu/inflammation/vomiting/abdominal pain/general health problems)
- No retained sample or metabolite has 15% or more missing values after filtering
- The output matrix retains metabolite identifiers consistent with the input Mtblmcs values/variables tables
- The filtered matrix is the documented input for the subsequent sample-wise creatinine normalization step in the downstream action.nf workflow

## Limitations

- The exclusion criteria are urine- and cohort-specific (ACTION Biomarker Study NTR and LUMC-Curium cohorts) and cannot be applied without matching dipstick, collection-timing, and health-report metadata
- Only synthetic data can be shared publicly because privacy regulations prohibit sharing personally identifiable omics measurements, limiting external validation on the real cohort
- Running the workflow requires Nextflow plus Singularity/Docker container infrastructure, which may require initial training of researchers

## Evidence

- [discussion] Exclude subjects who have started menstruating: "we excluded samples from the metabolomics data from (1) subjects who have started menstruating,"
- [discussion] Exclude >2h storage delay: "subjects in whom the time between urine sample collection and storing in the freezer was >2 hours,"
- [discussion] Dipstick criteria: "subjects in whom the leukocyte count was above trace, (5) subjects in whom the nitrites level was “positive high,” (6) subjects in whom the protein level was >0.3,"
- [discussion] 15% missingness threshold: "Samples and metabolites with 15% or more of missing values were discarded."
- [readme] Preprocessing repo purpose: [evidence span withheld — anchor status: not_found]
