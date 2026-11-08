---
name: ewas-atlas-trait-enrichment-analysis
description: Use when MOFA2 factors computed on Infinium MethylationEPIC DNA methylation
  data need biological interpretation via CpG-trait enrichment in the EWAS atlas.
when_to_use_negative:
- Factors come from the metabolomics or behavioral (MCA) views rather than the methylation
  view — the skill operates on methylation CpG weights against EWAS atlas CpG sets
- The goal is differential methylation testing between sample groups rather than unsupervised
  factor annotation via trait-set over-representation
- No Infinium MethylationEPIC annotation or EWAS atlas catalogue is available, so
  probes cannot be mapped to traits
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow manager that orchestrates the EWAS enrichment module and writes results
    to the --output directory
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime automatically pulled by Nextflow to provide the module's
    software dependencies
  repo: https://github.com/Xomics/Docker_containers.git
- name: MOFA2
  role: Produces the latent-factor weight matrix for the buccal DNA methylation view
    that provides the top-100 CpGs per factor
- name: EWAS atlas
  role: Reference catalogue of CpG-trait associations against which each factor's
    CpG set is over-representation tested
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:c8f9044cb561a40a705eab357f751ac338719319efade732f810c9b38f705205
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - EWAS atlas
  - MOFA2
  - Nextflow
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ewas-atlas-trait-enrichment-analysis

## Summary

Annotates methylation latent factors from the ACTIONdemonstrator workflow by over-representation testing of each MOFA factor's top-weighted CpGs against trait-specific CpG sets in the EWAS atlas, recovering factor-level trait associations such as glucocorticoid exposure, aging, and immune-related traits.

## When to use

Use when MOFA2 factors computed on Infinium MethylationEPIC DNA methylation data need biological interpretation via CpG-trait enrichment in the EWAS atlas.

## When NOT to use

- Factors come from the metabolomics or behavioral (MCA) views rather than the methylation view — the skill operates on methylation CpG weights against EWAS atlas CpG sets
- The goal is differential methylation testing between sample groups rather than unsupervised factor annotation via trait-set over-representation
- No Infinium MethylationEPIC annotation or EWAS atlas catalogue is available, so probes cannot be mapped to traits

## Inputs

- MOFA2 latent-factor weight matrix for the buccal DNA methylation view (per-factor CpG weights)
- Infinium MethylationEPIC product annotation file (Illumina product file; placeholder in EPIC_annotation/raw/)
- EWAS atlas catalogue of CpG-trait associations

## Outputs

- Factor x trait enrichment table with overlap-size and FDR-corrected significance columns, written to the Nextflow --output directory

## How to apply

Load the latent-factor weight matrix for the buccal DNA methylation view computed upstream with MOFA2 (R/Bioconductor MOFA2 1.3.4). For each methylation factor, rank CpG probes by the magnitude of their MOFA weights and retain the top 100 CpGs per factor. Map the selected probe identifiers to traits using the Infinium MethylationEPIC product annotation together with the EWAS atlas catalogue of CpG-trait associations. Perform over-representation testing (hypergeometric / Fisher's exact test) of each factor's CpG set against trait-specific CpG sets in the EWAS atlas, recording overlap size and nominal p-value for every factor-trait pair, then correct p-values for multiple testing across all factor-trait combinations with FDR. Assemble the results into a factor x trait enrichment table with overlap-size and significance columns and write it to the Nextflow --output directory; note that the upstream factor analysis only includes the top 10% most variable probes from the 787,711 autosomal methylation probes that survived QC.

## Related tools

- **Nextflow** (Workflow manager that orchestrates the EWAS enrichment module and writes results to the --output directory) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime automatically pulled by Nextflow to provide the module's software dependencies) — https://github.com/Xomics/Docker_containers.git
- **MOFA2** (Produces the latent-factor weight matrix for the buccal DNA methylation view that provides the top-100 CpGs per factor)
- **EWAS atlas** (Reference catalogue of CpG-trait associations against which each factor's CpG set is over-representation tested)

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv
```

## Evaluation signals

- The enrichment table file exists in the specified --output directory
- The table contains one entry per factor-trait pair with populated overlap-size and significance (FDR) fields
- Methylation factors recover trait enrichments consistent with the reported glucocorticoid, aging, and immune associations
- Each factor's CpG set contains exactly the top 100 CpGs by MOFA weight magnitude
- Enrichment p-values are FDR-corrected across all factor-trait combinations, not nominal

## Limitations

- Enrichment is limited to the top 100 CpGs per factor, so weakly weighted but trait-relevant probes are not considered
- Upstream only the top 10% most variable probes from the 787,711 autosomal methylation probes that survived QC enter the analysis, restricting which CpGs can be enriched
- The workflow requires downloading the missing Infinium MethylationEPIC product file from Illumina; a placeholder is in EPIC_annotation/raw/ and storage limits mean the GitHub repository does not contain all required files (use the WorkflowHub RO-Crate instead)

## Evidence

- [methods] The factor-annotation module selects, for each MOFA factor, the top 100 CpGs with the largest weights and performs enrichment analysis within the EWAS atlas, reporting factor-level enrichments such as glucocorticoid exposure, aging, and immune-related traits.: [evidence span withheld — anchor status: not_found]
- [discussion] We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4)"
- [discussion] From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses.: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included"
- [readme] Then download the missing Infinium MethylationEPIC product file [here](https://emea.support.illumina.com/downloads/infinium-methylationepic-v1-0-product-files.html), for which a placeholder is located in `EPIC_annotation/raw/`.: "download the missing Infinium MethylationEPIC product file"
