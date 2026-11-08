---
name: metabolomics-data-integration
description: Use when two standardized omics views — e.g. creatinine-normalized urine
  metabolomics intensities and top-10%-variable autosomal buccal DNA methylation probes
  — need to be integrated into shared latent factors without supervision.
when_to_use_negative:
- The goal is to factorize behavioral/phenotype data (e.g. CBCL items) — the paper
  uses Multiple Correspondence Analysis via R library FactoMineR (2.4) for that, not
  MOFA.
- The metabolomics data are still raw urine intensities without sample-wise creatinine
  normalization or the 15%-missing-value filtering — the workflow expects preprocessed,
  standardized input.
- A supervised or phenotype-driven analysis is needed (e.g. testing behavioral dimension
  scores across SNF clusters) — this skill is purely unsupervised latent-factor decomposition.
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MOFA2
  role: R/Bioconductor library (v1.3.4) that trains the 10-factor multi-view model
    on the DNAm and metabolomics views
- name: Nextflow
  role: Workflow manager that runs the action.nf module encapsulating the MOFA step
    with --output, --mtblmcs_values and --epigenomics_values parameters
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime whose MOFA2 image (pulled from Dockerhub) captures the software
    dependencies of the integration step
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:e9f7454c701e5d34e0c02783e6f23931c1624a668210df008940cd195f2b25b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolomics-data-integration

## Summary

Jointly decomposes two omics views — urine metabolomics and buccal DNA methylation — into 10 shared latent factors with MOFA2 (R/Bioconductor 1.3.4), executed as a Nextflow/Docker module of the ACTIONdemonstrator workflow, and exports factor values plus per-view variance explained.

## When to use

Use when two standardized omics views — e.g. creatinine-normalized urine metabolomics intensities and top-10%-variable autosomal buccal DNA methylation probes — need to be integrated into shared latent factors without supervision.

## When NOT to use

- The goal is to factorize behavioral/phenotype data (e.g. CBCL items) — the paper uses Multiple Correspondence Analysis via R library FactoMineR (2.4) for that, not MOFA.
- The metabolomics data are still raw urine intensities without sample-wise creatinine normalization or the 15%-missing-value filtering — the workflow expects preprocessed, standardized input.
- A supervised or phenotype-driven analysis is needed (e.g. testing behavioral dimension scores across SNF clusters) — this skill is purely unsupervised latent-factor decomposition.

## Inputs

- Creatinine-normalized urine metabolomics matrix (CSV/TSV, samples x metabolites) filtered for missing values, e.g. Synthetic_data/synthetic_metabolomics.csv
- DNA methylation beta-value matrix (samples x probes, CSV) restricted to the top 10% most variable autosomal probes, e.g. Synthetic_data/synthetic_epigenomics.csv
- Nextflow module action.nf with MOFA2 Docker container

## Outputs

- Tab-separated factor matrix (samples x 10 factors)
- Variance-explained table reporting R2 per factor per view (DNAm and metabolomics)

## How to apply

First restrict the DNA methylation beta matrix (e.g. Synthetic_data/synthetic_epigenomics.csv) to the top 10% most variable autosomal probes by ranking on variance, and use the urine metabolomics matrix (e.g. Synthetic_data/synthetic_metabolomics.csv) that has been creatinine-normalized sample-wise and filtered for missing values as the second view. Then run the Nextflow module action.nf of the ACTIONdemonstrator workflow, which executes R/Bioconductor MOFA2 (v1.3.4) inside its Docker container to train a multi-view factor model with 10 factors on the two views. The module exports a samples-by-10-factors tab-separated factor matrix and a variance-explained (R2 per factor per view) table for DNAm and metabolomics. The rationale is that MOFA learns cross-view structure while attributing each factor's R2 per view, letting you judge whether a factor is methylation- or metabolomics-driven; in the paper's cohorts the cumulative per-view R2 was 22.5% (DNAm) vs 0.001% (metabolomics) in NTR and 74.9% (DNAm) vs 1.89% (metabolomics) in LUMC-Curium.

## Related tools

- **MOFA2** (R/Bioconductor library (v1.3.4) that trains the 10-factor multi-view model on the DNAm and metabolomics views)
- **Nextflow** (Workflow manager that runs the action.nf module encapsulating the MOFA step with --output, --mtblmcs_values and --epigenomics_values parameters) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime whose MOFA2 image (pulled from Dockerhub) captures the software dependencies of the integration step) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv
```

## Evaluation signals

- Both output files exist in the --output directory: the factor matrix and the variance-explained-per-view table.
- The factor matrix contains exactly 10 factor columns (one per trained factor).
- The variance-explained table reports an R2 value for each of the two views (DNAm and metabolomics) per factor.
- Per-view cumulative R2 falls in plausible ranges — e.g. NTR: 22.5% (DNAm) and 0.001% (metabolomics); LUMC-Curium: 74.9% (DNAm) and 1.89% (metabolomics).
- Input DNAm matrix was restricted to the top 10% most variable autosomal probes and the metabolomics matrix was creatinine-normalized and missing-value filtered before model training.

## Limitations

- The metabolomics view may explain almost no variance (0.001% in NTR), so factors can be effectively methylation-only despite the joint model.
- Privacy regulations prevent sharing the real omics/clinical data; only a synthetic dataset is provided for demonstrating functionality.
- No matching EDAM term was available for unsupervised learning steps (eNanoMapper Ontology had to be used), complicating semantic annotation of this skill in RO-Crate.
- MOFA2 quality depends on upstream preprocessing (probe variance filtering, creatinine normalization, 15% missing-value cutoff); running it on unfiltered inputs is out of scope for this module.

## Evidence

- [discussion] We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [discussion] From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses.: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [discussion] Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels: "Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels"
- [readme] Synthetic data is provided as example data in `Synthetic_data/`: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [readme] The typical command to run the workflow is: nextflow run action.nf --output dir/of/choice: "The typical command to run the workflow is:
```
nextflow run action.nf 
            --output dir/of/choice
            --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv"
