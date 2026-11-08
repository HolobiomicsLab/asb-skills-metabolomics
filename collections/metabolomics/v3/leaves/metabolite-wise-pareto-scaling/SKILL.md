---
name: metabolite-wise-pareto-scaling
description: Use when preprocessing urine metabolite intensity tables (e.g. synthetic_metabolomics.csv)
  for downstream integration, where metabolites differ in variance and must be scaled
  before MOFA or similarity network analysis.
when_to_use_negative:
- The metabolomics matrix is already creatinine-normalized and Pareto-scaled (re-scaling
  would alter validated downstream inputs)
- Data are DNA methylation probes, which undergo a different QC route (top 10% most
  variable autosomal probes) rather than creatinine normalization and Pareto scaling
- Sample-wise creatinine normalization has not yet been performed, since Pareto scaling
  is applied after dilution correction in this workflow
tools:
- name: Nextflow
  role: Workflow manager orchestrating the metabolomics preprocessing process (QC,
    creatinine normalization, Pareto scaling) in action.nf
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Provides the containerized software dependencies that Nextflow pulls automatically
    for the preprocessing steps
  repo: https://github.com/Xomics/Docker_containers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:a579c9595e38aad20b37ff3bc0248a8e1d22bcc45b6363aaa86da3e3a6fd99bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-wise-pareto-scaling

## Summary

A metabolomics preprocessing step in the ACTIONdemonstrator workflow that scales each urine metabolite to mean-zero with variance adjusted by the square root of its standard deviation (Pareto scaling), applied after sample-wise creatinine normalization to make metabolites comparable in downstream multi-omics integration.

## When to use

Use when preprocessing urine metabolite intensity tables (e.g. synthetic_metabolomics.csv) for downstream integration, where metabolites differ in variance and must be scaled before MOFA or similarity network analysis.

## When NOT to use

- The metabolomics matrix is already creatinine-normalized and Pareto-scaled (re-scaling would alter validated downstream inputs)
- Data are DNA methylation probes, which undergo a different QC route (top 10% most variable autosomal probes) rather than creatinine normalization and Pareto scaling
- Sample-wise creatinine normalization has not yet been performed, since Pareto scaling is applied after dilution correction in this workflow

## Inputs

- Metabolite intensity table (CSV, e.g. Synthetic_data/synthetic_metabolomics.csv passed via --mtblmcs_values)
- Sample creatinine levels for sample-wise normalization
- Urine-quality/protocol QC criteria for sample exclusion

## Outputs

- QC-filtered, creatinine-normalized, Pareto-scaled metabolomics matrix (preprocessed_metabolomics.csv in the workflow output directory)

## How to apply

In the ACTIONdemonstrator Nextflow workflow (action.nf), the metabolomics matrix is first filtered: sample-level QC exclusions (e.g. menstruation, >2 h delay to freezer, severe protocol violations, abnormal urinalysis leukocyte/nitrite/protein/glucose/blood, illness) are applied, and samples and metabolites with 15% or more missing values are discarded. Next, sample-wise normalization corrects for urine concentration by adjusting metabolite intensities to each sample's creatinine levels. Then, metabolite-wise Pareto scaling is applied so that each metabolite is scaled to mean-zero with variance adjusted by the square root of its standard deviation — reducing large-variance metabolites' dominance while preserving relative intensity differences. The QC-filtered, creatinine-normalized, Pareto-scaled matrix is written to preprocessed_metabolomics.csv in the workflow output directory for downstream MOFA2 and SNF analysis.

## Related tools

- **Nextflow** (Workflow manager orchestrating the metabolomics preprocessing process (QC, creatinine normalization, Pareto scaling) in action.nf) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Provides the containerized software dependencies that Nextflow pulls automatically for the preprocessing steps) — https://github.com/Xomics/Docker_containers

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv
```

## Evaluation signals

- preprocessed_metabolomics.csv exists in the workflow output directory after the run
- Output table retains metabolite identifiers as columns/rows
- No retained sample or metabolite exceeds the 15% missingness threshold
- Each scaled metabolite column has (approximately) mean zero after Pareto scaling
- Sample-wise creatinine normalization was applied before scaling (dilution correction is reflected in the intensity ratios)

## Limitations

- Real omics/clinical data cannot be shared due to privacy regulations; the workflow is demonstrated only with synthetic data
- Scaling is applied after the specific ACTION Biomarker Study urine QC exclusions; reuse requires adapting the QC criteria to other cohorts
- Running locally requires additional files (e.g. the Infinium MethylationEPIC product file) not stored in the git repository; the complete RO-Crate must be downloaded from WorkflowHub

## Evidence

- [discussion] Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels: "Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels"
- [discussion] Samples and metabolites with 15% or more of missing values were discarded.: "Samples and metabolites with 15% or more of missing values were discarded."
- [readme] Synthetic data example files in repository: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [full_text] Pareto scaling step from task workflow: [evidence span withheld — anchor status: not_found]
