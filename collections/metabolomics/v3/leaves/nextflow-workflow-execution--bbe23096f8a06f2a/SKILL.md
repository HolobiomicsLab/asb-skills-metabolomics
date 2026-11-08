---
name: nextflow-workflow-execution
description: Use when a multi-omics analysis (e.g. metabolomics CSV, methylation betas,
  CBCL behavioral items) must be run end-to-end through a reproducible, containerized
  Nextflow pipeline rather than ad-hoc scripts.
when_to_use_negative:
- The raw cohort data has not yet been converted to the standardized file formats
  expected by action.nf — the separate ACTIONdemonstrator_preprocessing workflow (action_file_preparation.nf)
  handles that standardization step
- Only a single isolated step (e.g. one imputation or one scaling) is needed outside
  the packaged pipeline, and container/Nextflow overhead is not justified
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: workflow manager that runs the modular action.nf pipeline (v22.04.0 installable
    via bioconda), orchestrating all preprocessing and analysis processes
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: provides containerized software dependencies; containers are pulled automatically
    from Dockerhub via the default nextflow.config
  repo: https://github.com/Xomics/Docker_containers.git
- name: Singularity/Apptainer
  role: alternative container runtime used in the Digital Research Environment via
    the dre.config file and locally built .sif files
  repo: https://github.com/Xomics/Docker_containers.git
- name: WorkflowHub / RO-Crate
  role: registers and packages the workflow (workflow 402) with rich semantic metadata
    as a FAIR Digital Object
  repo: https://workflowhub.eu/workflows/402
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:9ad0e8d151649346e3adab00304e83180eec6c8598b2167301ef9b67679bfe4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  - Singularity/Apptainer
  - WorkflowHub / RO-Crate
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# nextflow-workflow-execution

## Summary

Execute the ACTIONdemonstrator multi-omics analysis pipeline (metabolomics preprocessing, MOFA, MCA, SNF) as a modular Nextflow pipeline with containerized software dependencies, so that the packaged workflow is reproducible and reusable as a FAIR Digital Object.

## When to use

Use when a multi-omics analysis (e.g. metabolomics CSV, methylation betas, CBCL behavioral items) must be run end-to-end through a reproducible, containerized Nextflow pipeline rather than ad-hoc scripts.

## When NOT to use

- The raw cohort data has not yet been converted to the standardized file formats expected by action.nf — the separate ACTIONdemonstrator_preprocessing workflow (action_file_preparation.nf) handles that standardization step
- Only a single isolated step (e.g. one imputation or one scaling) is needed outside the packaged pipeline, and container/Nextflow overhead is not justified

## Inputs

- urine metabolomics values table (e.g. Synthetic_data/synthetic_metabolomics.csv)
- DNA methylation beta matrix (e.g. Synthetic_data/synthetic_epigenomics.csv) with epigenomics metadata CSV
- MAF files directory (e.g. Synthetic_data/)
- ASEBA CBCL behavioral item table (e.g. Synthetic_data/synthetic_cbcl_data.csv)
- phenotype covariates table (e.g. Synthetic_data/synthetic_phenotype_covariates_data.csv)
- CBCL item labels (Synthetic_data/cbcl_labels.csv)
- subject ID crosswalk (e.g. Synthetic_data/ACTIONdemonstrator_XOmics_IDS_synthetic.csv)
- Infinium MethylationEPIC v1.0 annotation product file

## Outputs

- workflow result files written to the --output directory (e.g. preprocessed_metabolomics.csv, MOFA factor matrix and variance-explained table, imputed_cbcl_data.csv)
- optionally an RO-Crate .zip containing all workflow elements plus semantic metadata, downloadable from WorkflowHub workflow 402

## How to apply

Clone or download the workflow (git clone https://github.com/Xomics/ACTIONdemonstrator_workflow.git, or fetch the RO-Crate from WorkflowHub workflow 402, which contains all files that the repository omits due to storage limits). Run `nextflow run action.nf --help` for usage, then invoke with `--output` plus the concrete inputs from Synthetic_data/: synthetic_metabolomics.csv, synthetic_epigenomics.csv (plus epigenomics meta), synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, and the synthetic IDs file; before first use, download the missing Infinium MethylationEPIC v1.0 product file whose placeholder sits in EPIC_annotation/raw/. Nextflow automatically pulls all required Docker containers from Dockerhub via the default nextflow.config; in the Digital Research Environment (mydre.org) use `-c dre.config` and point `--container_dir` to local Singularity (.sif) files. The required run environment is Nextflow 22.04.0 (installable via conda/bioconda) plus Singularity or Docker; the RO-Crate packaging (generate_ro-crate.ipynb) makes the workflow findable and semantically annotated on WorkflowHub.

## Related tools

- **Nextflow** (workflow manager that runs the modular action.nf pipeline (v22.04.0 installable via bioconda), orchestrating all preprocessing and analysis processes) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (provides containerized software dependencies; containers are pulled automatically from Dockerhub via the default nextflow.config) — https://github.com/Xomics/Docker_containers.git
- **Singularity/Apptainer** (alternative container runtime used in the Digital Research Environment via the dre.config file and locally built .sif files) — https://github.com/Xomics/Docker_containers.git
- **WorkflowHub / RO-Crate** (registers and packages the workflow (workflow 402) with rich semantic metadata as a FAIR Digital Object) — https://workflowhub.eu/workflows/402

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- Workflow completes with Nextflow 22.04.0 and all containers resolve (Dockerhub pull via default nextflow.config, or local .sif via dre.config).
- Expected output files exist in the --output directory with correct content, e.g. the metabolomics output retains metabolite identifiers and no retained sample or metabolite exceeds the 15% missingness threshold.
- The RO-Crate downloaded from WorkflowHub (workflowhub.eu/workflows/402) contains all files needed to run, since the git repository omits some files due to storage limits.
- The Infinium MethylationEPIC product file is present in EPIC_annotation/raw/ (replacing the placeholder) before running.
- Downstream module validation passes, e.g. MOFA factor and variance-explained tables each contain 10 factors with R2 reported for both DNAm and metabolomics views.

## Limitations

- The git repository does not contain all files needed to run because of storage limits; the complete set must come from the WorkflowHub RO-Crate, and the Infinium MethylationEPIC product file must be downloaded separately from Illumina.
- Sensitive omics and clinical data cannot be shared due to privacy regulations; only synthetic data (Synthetic_data/) is provided for demonstration, so real-cohort behavior must be validated separately.
- Adopting workflow managers and containers may require initial training of researchers, and the DRE environment requires a specific config (-c dre.config) with a --container_dir of .sif files.

## Evidence

- [abstract] The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies.: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
- [intro] portable software containers: "portable software containers such as Apptainer/Singularity [33] or Docker [34] that capture the runtime environment of a workflow or a workflow module"
- [readme] Docker containers pulled automatically: "Nextflow automatically pulls all necessary Docker containers from Dockerhub, when using the default `nextflow.config` file."
- [discussion] tools well documented and quickly adopted: "we experienced that the tools were generally well documented and could quickly be adopted."
