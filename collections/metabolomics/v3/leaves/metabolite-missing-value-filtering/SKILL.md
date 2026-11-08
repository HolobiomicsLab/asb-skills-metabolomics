---
name: metabolite-missing-value-filtering
description: Use when preprocessing urine metabolite intensity tables (e.g. synthetic_metabolomics.csv
  in the ACTIONdemonstrator Nextflow pipeline) to remove samples or metabolites with
  excessive missingness prior to normalization and scaling.
when_to_use_negative:
- Input is already a QC-filtered, normalized, Pareto-scaled metabolomics matrix (e.g.
  preprocessed_metabolomics.csv) — re-filtering would double-apply the threshold
- 'Data type is DNA methylation: the analogous QC step there is filtering to the top
  10% most variable autosomal probes, not a 15% missingness cutoff'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Nextflow
  role: Workflow manager executing the preprocessing pipeline (action.nf / action_file_preparation.nf)
    that implements the missingness filter as a modular process
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containers pulled from Dockerhub by Nextflow provide the software dependencies
    for the filtering process
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:b07290abab0235c3794548e3c8b349d91d2d2e54cda63e221844626190c2d6a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-missing-value-filtering

## Summary

Quality-control step from the ACTIONdemonstrator multi-omics workflow that discards urine metabolomics samples and metabolites in which 15% or more of the intensity values are missing, before creatinine normalization and Pareto scaling.

## When to use

Use when preprocessing urine metabolite intensity tables (e.g. synthetic_metabolomics.csv in the ACTIONdemonstrator Nextflow pipeline) to remove samples or metabolites with excessive missingness prior to normalization and scaling.

## When NOT to use

- Input is already a QC-filtered, normalized, Pareto-scaled metabolomics matrix (e.g. preprocessed_metabolomics.csv) — re-filtering would double-apply the threshold
- Data type is DNA methylation: the analogous QC step there is filtering to the top 10% most variable autosomal probes, not a 15% missingness cutoff

## Inputs

- Metabolomics intensity table (CSV, e.g. Synthetic_data/synthetic_metabolomics.csv) with samples as rows/columns and metabolite identifiers
- Sample-level QC metadata (urine-quality and protocol-violation criteria from the ACTION Biomarker Study cohorts NTR-ACTION and LUMC-Curium)

## Outputs

- QC-filtered metabolomics matrix retaining only samples and metabolites with <15% missing values (later written as preprocessed_metabolomics.csv after normalization and scaling)

## How to apply

Load the metabolomics intensity table (e.g. Synthetic_data/synthetic_metabolomics.csv, as expected by the Nextflow process in action.nf) and compute the per-sample and per-metabolite missingness rate. Apply the sample-level QC exclusions first (menstruation, >2 h to freezer storage, sampling-protocol violations, urinalysis flags, illness reports), then discard any sample or metabolite for which 15% or more of the values are missing. This threshold is applied before sample-wise creatinine normalization and metabolite-wise Pareto scaling so that normalization statistics are computed only on sufficiently complete data. Write the filtered matrix onward in the workflow (the full pipeline emits preprocessed_metabolomics.csv after the subsequent normalization/scaling steps).

## Related tools

- **Nextflow** (Workflow manager executing the preprocessing pipeline (action.nf / action_file_preparation.nf) that implements the missingness filter as a modular process) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containers pulled from Dockerhub by Nextflow provide the software dependencies for the filtering process) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv
```

## Evaluation signals

- No retained sample or metabolite exceeds the 15% missingness threshold in the filtered matrix
- Metabolite identifiers are preserved as columns/rows of the output table (schema check on preprocessed_metabolomics.csv)
- Rows removed correspond exactly to samples failing the urinalysis/protocol QC criteria or the 15% missingness rule
- Filtering occurs upstream of creatinine normalization and Pareto scaling, so downstream scaling statistics are computed on the filtered data only
- Output file exists in the workflow output directory after the Nextflow run completes

## Limitations

- The threshold (15%) is cohort-specific to the ACTION Biomarker Study urine metabolomics data; applying it to sparser metabolomics platforms may discard too much data
- Real (potentially personally identifiable) omics data cannot be shared due to privacy regulations, so the filter is demonstrated on synthetic data only
- The filter depends on upstream sample-level QC criteria (menstruation, >2 h freezer delay, protocol violations, urinalysis flags, illness reports) that require phenotype metadata not present in the intensity table itself

## Evidence

- [discussion] 15% missingness threshold (discussion): "Samples and metabolites with 15% or more of missing values were discarded."
- [discussion] upstream QC exclusions (discussion): "we excluded samples from the metabolomics data from (1) subjects who have started menstruating,"
- [discussion] creatinine normalization follows filtering (discussion): "Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels"
- [readme] synthetic input data location (readme): "Synthetic data is provided as example data in `Synthetic_data/`."
- [readme] workflow input file (readme): "--mtblmcs_values Sythetic_data/synthetic_metabolomics.csv"
