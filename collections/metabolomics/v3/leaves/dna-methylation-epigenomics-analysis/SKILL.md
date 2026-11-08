---
name: dna-methylation-epigenomics-analysis
description: Use when annotating methylation latent factors (e.g. MOFA2 factors from
  MethylationEPIC beta matrices) with biological traits via EWAS atlas enrichment,
  or when planning CpG-level methylation analysis steps.
when_to_use_negative:
- Input omics data are metabolomics or behavioral (MCA) factors rather than DNA methylation
  factor weights — the CpG ranking and EWAS atlas mapping do not apply
- Methylation probes have not passed QC and variable-probe filtering (only the top
  10% most variable of 787,711 autosomal EPIC probes enter the analysis)
- The goal is differential methylation testing between sample groups rather than unsupervised
  factor-level trait annotation
edam_topics:
- http://edamontology.org/topic_3656
tools:
- name: MOFA2
  role: R/Bioconductor library (1.3.4) that computes the methylation latent factors
    whose weights are ranked for top-100 CpG selection
- name: Nextflow
  role: Workflow manager that orchestrates the methylation preprocessing, MOFA2 factor
    analysis, and EWAS enrichment step (action.nf)
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Provides the pinned software containers (from Xomics/Docker_containers) that
    encapsulate the R environment running MOFA2 and the enrichment testing
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:b9c3e9be0c7ee5dac062ed501214cd5067371677f45b045c000cb80fbabdebdb
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
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dna-methylation-epigenomics-analysis

## Summary

Epigenomic analysis of buccal Infinium MethylationEPIC DNA methylation data within the ACTIONdemonstrator workflow: after QC and inclusion of the top 10% most variable autosomal probes, MOFA2 latent factors are annotated by over-representation testing of each factor's top-100 CpGs against EWAS atlas trait associations.

## When to use

Use when annotating methylation latent factors (e.g. MOFA2 factors from MethylationEPIC beta matrices) with biological traits via EWAS atlas enrichment, or when planning CpG-level methylation analysis steps.

## When NOT to use

- Input omics data are metabolomics or behavioral (MCA) factors rather than DNA methylation factor weights — the CpG ranking and EWAS atlas mapping do not apply
- Methylation probes have not passed QC and variable-probe filtering (only the top 10% most variable of 787,711 autosomal EPIC probes enter the analysis)
- The goal is differential methylation testing between sample groups rather than unsupervised factor-level trait annotation

## Inputs

- MOFA2 latent-factor weight matrix for the buccal DNA methylation view (epigenomics values CSV, e.g. synthetic_epigenomics.csv)
- Infinium MethylationEPIC v1.0 product annotation file (placeholder in EPIC_annotation/raw/)
- EWAS atlas catalogue of CpG-trait associations
- Epigenomics sample metadata (synthetic_epigenomics_meta.csv)

## Outputs

- Factor x trait enrichment table with overlap-size and FDR-corrected significance columns, written to the Nextflow --output directory
- Per factor-trait pair nominal p-values and overlap counts

## How to apply

Load the latent-factor weight matrix for the buccal DNA methylation view computed with MOFA2 (R/Bioconductor MOFA2 1.3.4); methylation inputs must first have survived QC (787,711 autosomal probes, with only the top 10% most variable probes retained for analysis). For each methylation factor, rank CpG probes by the magnitude of their MOFA weights and retain the top 100 CpGs per factor. Map probe identifiers to traits using the Infinium MethylationEPIC product annotation together with the EWAS atlas catalogue of CpG-trait associations, then run over-representation testing (hypergeometric / Fisher's exact) of each factor's CpG set against trait-specific CpG sets, recording overlap size and nominal p-value for every factor-trait pair. Correct p-values for multiple testing across all factor-trait combinations with FDR, and assemble the factor x trait enrichment table into the Nextflow --output directory.

## Related tools

- **MOFA2** (R/Bioconductor library (1.3.4) that computes the methylation latent factors whose weights are ranked for top-100 CpG selection)
- **Nextflow** (Workflow manager that orchestrates the methylation preprocessing, MOFA2 factor analysis, and EWAS enrichment step (action.nf)) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Provides the pinned software containers (from Xomics/Docker_containers) that encapsulate the R environment running MOFA2 and the enrichment testing) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv
```

## Evaluation signals

- Enrichment table file exists in the Nextflow --output directory
- Table contains one entry per factor-trait pair with populated overlap-size and significance fields
- Methylation factors recover trait enrichments consistent with the reported glucocorticoid exposure, aging, and immune-related associations
- Top-100 CpG sets derive from |MOFA weight| ranking of the methylation factors, not from arbitrary probe subsets
- Enrichment p-values are FDR-corrected across all factor-trait combinations, not reported as nominal only

## Limitations

- Privacy regulations prohibit sharing the real omics measurements; only a synthetic dataset (Synthetic_data/) is distributed, so results are demonstrative
- The Infinium MethylationEPIC product file must be downloaded manually from Illumina because storage limits keep it out of the git repository
- Only the top 10% most variable probes of the 787,711 QC-surviving autosomal probes are analyzed, limiting sensitivity to less-variable CpGs

## Evidence

- [methods] The factor-annotation module selects, for each MOFA factor, the top 100 CpGs with the largest weights and performs enrichment analysis within the EWAS atlas: [evidence span withheld — anchor status: not_found]
- [discussion] From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses.: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [discussion] We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4)"
- [methods] download the missing Infinium MethylationEPIC product file: "download the missing Infinium MethylationEPIC product file"
- [readme] nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv: "nextflow run action.nf 
            --output dir/of/choice
            --epigenomics_values Synthetic_data/synthetic_epigenomics.csv"
