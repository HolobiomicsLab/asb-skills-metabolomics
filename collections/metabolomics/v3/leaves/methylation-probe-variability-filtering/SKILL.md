---
name: methylation-probe-variability-filtering
description: Use when a buccal DNA methylation matrix (Infinium MethylationEPIC betas)
  must be reduced to its most variable autosomal probes as a preprocessing view for
  unsupervised multi-omics integration.
when_to_use_negative:
- The methylation matrix is the input for probe-level QC or annotation (e.g. retrieving
  the Infinium MethylationEPIC product file) rather than feature selection prior to
  factor analysis.
- The downstream method requires all probes or a fixed CpG panel rather than a variability-ranked
  subset (e.g. replication on preselected DMRs).
- Data are not methylation betas (e.g. the urine metabolomics view, which is instead
  filtered by 15% missing-value thresholds and creatinine normalization).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3518
tools:
- name: Nextflow
  role: Workflow manager executing the modular pipeline module that performs the top-10%
    most-variable-probe filtering step on the methylation matrix before MOFA2 training.
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime packaging the software dependencies of the filtering and
    downstream MOFA2 steps for reproducibility.
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:8bd97ae291a36865231541b204229e2ffa364aa8cde2df52f3aeba85cf216419
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# methylation-probe-variability-filtering

## Summary

Reduces a DNA methylation beta-value matrix to the top 10% most variable autosomal probes before multi-omics factor analysis, so that MOFA2 runs on a tractable, information-rich probe set rather than the full 787,711-probe QC-surviving EPIC array.

## When to use

Use when a buccal DNA methylation matrix (Infinium MethylationEPIC betas) must be reduced to its most variable autosomal probes as a preprocessing view for unsupervised multi-omics integration.

## When NOT to use

- The methylation matrix is the input for probe-level QC or annotation (e.g. retrieving the Infinium MethylationEPIC product file) rather than feature selection prior to factor analysis.
- The downstream method requires all probes or a fixed CpG panel rather than a variability-ranked subset (e.g. replication on preselected DMRs).
- Data are not methylation betas (e.g. the urine metabolomics view, which is instead filtered by 15% missing-value thresholds and creatinine normalization).

## Inputs

- QC-filtered DNA methylation beta-value matrix restricted to autosomal probes (e.g. Synthetic_data/synthetic_epigenomics.csv; 787,711 autosomal probes after QC)
- Sample identifiers matching the methylation matrix to the other omics/phenotype views (e.g. ACTIONdemonstrator_XOmics_IDS_synthetic.csv)

## Outputs

- Reduced DNA methylation matrix containing only the top 10% most variable autosomal probes, suitable as the DNAm view for MOFA2

## How to apply

Start from the QC-filtered DNA methylation matrix of autosomal Infinium MethylationEPIC probes (787,711 probes survived QC in the ACTION case study), loaded e.g. from Synthetic_data/synthetic_epigenomics.csv. Rank probes by their variance across samples and retain only the top 10% most variable probes, discarding the remainder. This variability-based selection precedes MOFA2 factor extraction so that the downstream 10-factor model is not diluted by thousands of near-invariant CpG sites. The reduced matrix then serves as the DNAm view alongside the creatinine-normalized, missing-value-filtered urine metabolomics matrix when the Nextflow workflow action.nf is executed. Note that this step is applied per analysis view; it complements, but does not replace, upstream methylation array QC and annotation steps (the Infinium MethylationEPIC product file is needed elsewhere in the workflow).

## Related tools

- **Nextflow** (Workflow manager executing the modular pipeline module that performs the top-10% most-variable-probe filtering step on the methylation matrix before MOFA2 training.) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime packaging the software dependencies of the filtering and downstream MOFA2 steps for reproducibility.) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- The retained probe set equals ~10% of the autosomal probes that survived QC (paper context: top 10% of 787,711 autosomal probes).
- No probes on non-autosomal chromosomes remain in the filtered matrix.
- Probe variance ranking is computed within-cohort so the selection reflects that cohort's sample set.
- The filtered matrix feeds the MOFA2 DNAm view and the variance-explained table reports an R2 value for the DNAm view per factor (paper reported 22.5% in NTR and 74.9% in LUMC-Curium per view).
- Output files in the --output directory are complete and traceable as part of the FAIR workflow/RO-Crate run.

## Limitations

- Filtering is based on within-cohort variance, so the selected probe set is cohort-dependent and may not generalize to other cohorts.
- The real omics data cannot be shared due to privacy regulations; only synthetic example data ship in Synthetic_data/ for demonstrating the workflow.
- Running the full workflow requires the Infinium MethylationEPIC product file, which must be downloaded separately from Illumina (a placeholder sits in EPIC_annotation/raw/).

## Evidence

- [discussion] From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses.: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [other] Load the DNA methylation matrix and restrict it to the top 10% most variable autosomal probes: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic data is provided as example data in `Synthetic_data/`: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
