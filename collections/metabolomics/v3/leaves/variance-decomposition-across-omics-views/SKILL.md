---
name: variance-decomposition-across-omics-views
description: Use when two preprocessed omics matrices from the same samples (e.g.
  DNA methylation beta values and urine metabolite intensities) need shared latent
  factors and a per-view variance-explained breakdown.
when_to_use_negative:
- Only one omics view is available — MOFA2 multi-view modeling requires the two views
  (DNAm and metabolomics) from the same samples
- The goal is to obtain factors from behavioral/phenotype data — the paper uses MCA
  via the R library FactoMineR (2.4) for behavioral data, not MOFA
- Metabolomics data have not yet been creatinine-normalized and filtered for missing
  values (samples/metabolites with ≥15% missing values discarded) — preprocessing
  must happen first
tools:
- name: MOFA2
  role: R/Bioconductor library (v1.3.4) that trains the multi-view factor model on
    the DNAm and metabolomics views and computes R2 variance explained per factor
    per view
- name: Nextflow
  role: Workflow manager that runs the MOFA module via action.nf
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container that packages MOFA2 and its R/Bioconductor dependencies; pulled
    automatically from Dockerhub via nextflow.config, or run as Singularity .sif files
    in the DRE
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:fa0567467a112830f0a569372d34e94b673f4633fca2b6f9ca3e22016b6e603c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# variance-decomposition-across-omics-views

## Summary

Train a multi-view factor analysis model (R/Bioconductor MOFA2 v1.3.4) that jointly decomposes buccal DNA methylation and creatinine-normalized urine metabolomics data into 10 latent factors, exporting factor values and variance-explained (R2) per view per factor for each cohort.

## When to use

Use when two preprocessed omics matrices from the same samples (e.g. DNA methylation beta values and urine metabolite intensities) need shared latent factors and a per-view variance-explained breakdown.

## When NOT to use

- Only one omics view is available — MOFA2 multi-view modeling requires the two views (DNAm and metabolomics) from the same samples
- The goal is to obtain factors from behavioral/phenotype data — the paper uses MCA via the R library FactoMineR (2.4) for behavioral data, not MOFA
- Metabolomics data have not yet been creatinine-normalized and filtered for missing values (samples/metabolites with ≥15% missing values discarded) — preprocessing must happen first

## Inputs

- DNA methylation beta-value matrix (CSV, e.g. Synthetic_data/synthetic_epigenomics.csv) restricted to top 10% most variable autosomal probes
- Creatinine-normalized, missing-value-filtered urine metabolomics intensity matrix (CSV, e.g. Synthetic_data/synthetic_metabolomics.csv)
- Sample ID crosswalk (e.g. Synthetic_data/ACTIONdemonstrator_XOmics_IDS_synthetic.csv)

## Outputs

- Samples x 10 factors matrix exported as tab-separated values
- Variance-explained table (R2 per factor per view for DNAm and metabolomics)

## How to apply

Restrict the DNA methylation matrix (e.g. Synthetic_data/synthetic_epigenomics.csv) to the top 10% most variable autosomal probes by ranking probes on variance — from the 787,711 autosomal probes surviving QC the paper kept only the top 10%. Pair it with the creatinine-normalized, missing-value-filtered urine metabolomics matrix (e.g. Synthetic_data/synthetic_metabolomics.csv) as the second view. Run the Nextflow module `nextflow run action.nf --output dir/of/choice`, which executes MOFA2 (v1.3.4) inside its Docker container (Singularity .sif files in the DRE) to fit a multi-view factor model with 10 factors. Export the samples x 10 factors matrix as tab-separated values together with the R2-per-factor-per-view table. The paper reports cumulative variance explained per view of 22.5% (DNAm) and 0.001% (metabolomics) in NTR and 74.9% (DNAm) and 1.89% (metabolomics) in LUMC-Curium.

## Related tools

- **MOFA2** (R/Bioconductor library (v1.3.4) that trains the multi-view factor model on the DNAm and metabolomics views and computes R2 variance explained per factor per view)
- **Nextflow** (Workflow manager that runs the MOFA module via action.nf) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container that packages MOFA2 and its R/Bioconductor dependencies; pulled automatically from Dockerhub via nextflow.config, or run as Singularity .sif files in the DRE) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv
```

## Evaluation signals

- Both output files (factor matrix and variance-explained table) exist in the --output directory
- Factor matrix contains exactly 10 factors per sample
- Variance-explained table reports an R2 value for each of the two views (DNAm and metabolomics) per factor
- DNAm input was restricted to the top 10% most variable autosomal probes before training
- Paper reference values: cumulative variance explained per view of 22.5%/0.001% (NTR DNAm/metabolomics) and 74.9%/1.89% (LUMC-Curium DNAm/metabolomics)

## Limitations

- Privacy regulations prohibit sharing of the real personally identifiable omics measurements; only a synthetic dataset (Synthetic_data/) is distributed, so reproducible results differ from the published cohorts
- The metabolomics view explains very little variance (0.001% in NTR, 1.89% in LUMC-Curium), so factors are dominated by the DNAm view
- No exact EDAM ontology term was available to describe unsupervised learning workflow steps, which had to be tagged with the eNanoMapper Ontology in the RO-Crate

## Evidence

- [discussion] MOFA applied to buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [discussion] Top 10% most variable methylation probes filter: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [other] MOFA module outputs 10 factors with per-view R2: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic example data shipped with the workflow: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [readme] Docker containers pulled automatically by Nextflow: "Nextflow automatically pulls all necessary Docker containers from Dockerhub, when using the default `nextflow.config` file."
