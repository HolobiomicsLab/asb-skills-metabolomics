---
name: multi-omics-factor-analysis
description: Use when two or more standardized omics matrices (e.g. top-10%-most-variable
  DNAm beta values and creatinine-normalized urine metabolite intensities) need to
  be integrated into shared latent factors without a priori labels.
when_to_use_negative:
- Input matrices have not been preprocessed (DNAm not restricted to top 10% most variable
  autosomal probes, or metabolomics not creatinine-normalized and missing-value filtered)
- The goal is dimensionality reduction of categorical behavioral data — the paper
  uses MCA (FactoMineR 2.4) for behavioral data, not MOFA
- Only a single omics view is available; MOFA2 here is trained on two views (DNAm
  and metabolomics)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3339
tools:
- name: MOFA2
  role: R/Bioconductor library (v1.3.4) that trains the multi-view factor model with
    10 factors on the DNAm and metabolomics views
- name: Nextflow
  role: Workflow manager that runs the action.nf module wrapping MOFA2 in its container
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime providing the reproducible MOFA2 software environment pulled
    from Dockerhub via nextflow.config
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:b7682c1c0d82b13dbacb9b7627e0ab378fffd482302c35e6c9c4770c3fae235a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# multi-omics-factor-analysis

## Summary

Trains a multi-view unsupervised factor model with R/Bioconductor MOFA2 (v1.3.4), executed via the Nextflow module action.nf in a Docker container, to obtain 10 latent factors that jointly describe buccal DNA methylation and creatinine-normalized urine metabolomics data per cohort.

## When to use

Use when two or more standardized omics matrices (e.g. top-10%-most-variable DNAm beta values and creatinine-normalized urine metabolite intensities) need to be integrated into shared latent factors without a priori labels.

## When NOT to use

- Input matrices have not been preprocessed (DNAm not restricted to top 10% most variable autosomal probes, or metabolomics not creatinine-normalized and missing-value filtered)
- The goal is dimensionality reduction of categorical behavioral data — the paper uses MCA (FactoMineR 2.4) for behavioral data, not MOFA
- Only a single omics view is available; MOFA2 here is trained on two views (DNAm and metabolomics)

## Inputs

- DNA methylation beta-value matrix (samples x probes), e.g. Synthetic_data/synthetic_epigenomics.csv, restricted to top 10% most variable autosomal probes
- Preprocessed urine metabolomics matrix (creatinine-normalized, samples/metabolites with >=15% missing values discarded), e.g. Synthetic_data/synthetic_metabolomics.csv

## Outputs

- Factor values matrix (samples x 10 factors) as a tab-separated file in the --output directory
- Variance-explained-per-view table (R2 per factor per view: DNAm and metabolomics)

## How to apply

Load the DNA methylation matrix (e.g. Synthetic_data/synthetic_epigenomics.csv) restricted to the top 10% most variable autosomal probes by ranking probes on variance, and the creatinine-normalized, missing-value-filtered urine metabolomics matrix (e.g. Synthetic_data/synthetic_metabolomics.csv) as the second view. Run the Nextflow module (nextflow run action.nf --output dir/of/choice ...), which executes MOFA2 (v1.3.4) in its Docker container to train a multi-view factor model with 10 factors on the two omics views for each cohort (NTR and LUMC-Curium). Export the inferred factor values as a samples x 10 factors tab-separated matrix plus a variance-explained-per-view table (R2 per factor per view). In the paper this yielded cumulative variance explained of 22.5% (DNAm) and 0.001% (metabolomics) in NTR, and 74.9% (DNAm) and 1.89% (metabolomics) in LUMC-Curium. The resulting factors are subsequently correlated with behavioral MCA factors via Spearman's rank correlation.

## Related tools

- **MOFA2** (R/Bioconductor library (v1.3.4) that trains the multi-view factor model with 10 factors on the DNAm and metabolomics views)
- **Nextflow** (Workflow manager that runs the action.nf module wrapping MOFA2 in its container) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime providing the reproducible MOFA2 software environment pulled from Dockerhub via nextflow.config) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv
```

## Evaluation signals

- Both output files (factor matrix and variance-explained table) exist in the --output directory
- Factor matrix contains exactly 10 factors (samples x 10 factors)
- Variance-explained table reports an R2 value for each of the two views (DNAm and metabolomics) per factor
- Cumulative variance explained per view falls in plausible ranges comparable to the paper (e.g. 22.5%/74.9% for DNAm, 0.001%/1.89% for metabolomics in NTR/LUMC-Curium)
- Metabolomics view typically explains little variance, so very low metabolomics R2 values are expected rather than an error

## Limitations

- Privacy regulations prohibit sharing the real omics data; only synthetic data (Synthetic_data/) demonstrates the workflow, so absolute variance-explained values depend on the actual cohort data
- DNAm input is limited to the top 10% most variable autosomal probes (from 787,711 probes surviving QC); using unfiltered probes is outside the validated workflow
- The Infinium MethylationEPIC product file is not stored in the git repository (storage limits) and must be downloaded separately from Illumina

## Evidence

- [discussion] MOFA applied to buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [discussion] Top 10% most variable probes included: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [discussion] Creatinine normalization of metabolites: "Sample-wise normalization to correct for urine concentration was conducted by adjusting metabolite intensities to the sample creatinine levels"
- [readme] Typical workflow invocation: "nextflow run action.nf 
            --output dir/of/choice
            --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv
            --maf_files Synthetic_data/"
- [abstract] Modular Nextflow pipeline with containers: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
