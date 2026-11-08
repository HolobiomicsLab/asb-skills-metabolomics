---
name: mofa-factor-weight-extraction
description: Use when a fitted MOFA2 model of DNA methylation or metabolomics data
  needs its per-factor feature weight matrices extracted and ranked, e.g. to select
  top CpGs for enrichment analysis.
when_to_use_negative:
- The data are behavioral/phenotype variables — the workflow uses MCA (FactoMineR
  2.4), not MOFA, for those factors
- You need integrated multi-view similarity networks — use SNF/SNFpy clustering instead
  of factor weight extraction
- Only a single-omics dataset is available without a trained MOFA2 model — no factor
  weights exist to extract
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3173
tools:
- name: MOFA2
  role: R/Bioconductor library (1.3.4) used to fit the multi-omics factor model and
    from which the latent-factor weights are extracted
- name: Nextflow
  role: Workflow manager that runs the modular pipeline containing the MOFA2 factor-analysis
    and weight-extraction steps
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containerizes the software dependencies (incl. MOFA2) pulled automatically
    by Nextflow for the workflow processes
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:b38da0d4689a35f55684a9f5935f39c260e9ada706e3f056e9afa196470f7fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mofa-factor-weight-extraction

## Summary

Extract latent-factor weight matrices from MOFA2 (R/Bioconductor MOFA2 1.3.4) fitted to buccal DNA methylation and urine metabolomics views in the ACTIONdemonstrator workflow. These per-feature weights are the basis for downstream factor annotation, such as EWAS-atlas enrichment of top-weighted CpGs.

## When to use

Use when a fitted MOFA2 model of DNA methylation or metabolomics data needs its per-factor feature weight matrices extracted and ranked, e.g. to select top CpGs for enrichment analysis.

## When NOT to use

- The data are behavioral/phenotype variables — the workflow uses MCA (FactoMineR 2.4), not MOFA, for those factors
- You need integrated multi-view similarity networks — use SNF/SNFpy clustering instead of factor weight extraction
- Only a single-omics dataset is available without a trained MOFA2 model — no factor weights exist to extract

## Inputs

- Fitted MOFA2 model (MOFA2 1.3.4) trained on buccal DNA methylation (Infinium MethylationEPIC beta values, e.g. synthetic_epigenomics.csv) and urine metabolomics data
- Latent-factor weight matrix for the buccal DNA methylation view
- Infinium MethylationEPIC product annotation file (downloaded from Illumina; placeholder in EPIC_annotation/raw/)

## Outputs

- Per-factor ranked CpG lists (top 100 CpGs by |MOFA weight| per methylation factor)
- Latent-factor weight matrices per view available for downstream correlation and annotation steps

## How to apply

After the upstream MOFA2 factor analysis (MOFA2 1.3.4 applied to buccal DNA methylation and urine metabolomics data in the Nextflow ACTIONdemonstrator workflow), load the latent-factor weight matrix for the relevant view. For the methylation view, rank CpG probes by the magnitude of their MOFA weights and retain the top 100 CpGs per factor; these serve as input to EWAS-atlas over-representation testing. Note that methylation input is restricted to the 787,711 autosomal EPIC probes surviving QC, of which only the top 10% most variable probes were included in the analyses, so weights exist only for that probe subset. Rank by absolute weight magnitude so that factors with both positive and negative feature loadings are captured.

## Related tools

- **MOFA2** (R/Bioconductor library (1.3.4) used to fit the multi-omics factor model and from which the latent-factor weights are extracted)
- **Nextflow** (Workflow manager that runs the modular pipeline containing the MOFA2 factor-analysis and weight-extraction steps) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containerizes the software dependencies (incl. MOFA2) pulled automatically by Nextflow for the workflow processes) — https://github.com/Xomics/Docker_containers.git

## Examples

```
git clone https://github.com/Xomics/ACTIONdemonstrator_workflow.git && nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv
```

## Evaluation signals

- The methylation weight matrix covers only the top 10% most variable probes from the 787,711 autosomal EPIC probes that survived QC
- Each methylation factor yields exactly 100 CpGs when the top-weight selection is applied
- Downstream EWAS-atlas enrichment of the top-100 CpG sets recovers the reported glucocorticoid, aging, and immune trait associations
- The MOFA2 version is 1.3.4, matching the version reported for the ACTIONdemonstrator workflow

## Limitations

- Privacy regulations prohibit sharing the real omics data; only synthetic data in Synthetic_data/ is available, so weights from the public run derive from synthetic methylation values
- Only the top 10% most variable methylation probes enter the factor model, so weights for low-variance probes are absent
- The Infinium MethylationEPIC product file must be downloaded manually from Illumina (storage limits in the repository); a placeholder sits in EPIC_annotation/raw/

## Evidence

- [discussion] MOFA applied to buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [discussion] top 10% most variable probes: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [other] weight extraction rationale for top-100 CpG selection: [evidence span withheld — anchor status: not_found]
- [readme] workflow packaged in Nextflow: "Here, the ACTIONdemonstrator workflow is made using Nextflow workflow manager."
