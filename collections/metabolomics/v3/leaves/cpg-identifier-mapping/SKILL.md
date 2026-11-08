---
name: cpg-identifier-mapping
description: Use when annotating methylation latent factors (e.g. top-100 CpGs per
  MOFA factor) with biological traits, requiring CpG-to-trait reference sets from
  MethylationEPIC annotation and the EWAS atlas.
when_to_use_negative:
- The input omics view is urine metabolomics or behavioral data rather than DNA methylation
  probes — no CpG identifiers are present to map
- The analysis goal is clustering or factor extraction itself (MOFA/SNF/MCA steps)
  rather than post-hoc trait annotation of CpG sets
- The MethylationEPIC product annotation file has not been downloaded and only the
  placeholder file is present in EPIC_annotation/raw/
tools:
- name: MOFA2
  role: Supplies the latent-factor weight matrix from which the top 100 CpGs per methylation
    factor are ranked and selected
- name: EWAS atlas
  role: Reference catalogue of CpG-trait associations used to map probe identifiers
    and define trait-specific CpG sets for over-representation testing
- name: Nextflow
  role: Runs the factor-annotation module of the ACTIONdemonstrator pipeline (action.nf),
    writing the enrichment table to --output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:4ddbf7e3068da5cb75e7646089d2fc76f2cd101bed26dd55aadcc26786acd5f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - EWAS atlas
  - MOFA2
  - Nextflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cpg-identifier-mapping

## Summary

Maps selected CpG probe identifiers to biological traits by combining the Infinium MethylationEPIC product annotation with the EWAS atlas catalogue of CpG-trait associations, enabling factor-level trait enrichment in the ACTIONdemonstrator workflow.

## When to use

Use when annotating methylation latent factors (e.g. top-100 CpGs per MOFA factor) with biological traits, requiring CpG-to-trait reference sets from MethylationEPIC annotation and the EWAS atlas.

## When NOT to use

- The input omics view is urine metabolomics or behavioral data rather than DNA methylation probes — no CpG identifiers are present to map
- The analysis goal is clustering or factor extraction itself (MOFA/SNF/MCA steps) rather than post-hoc trait annotation of CpG sets
- The MethylationEPIC product annotation file has not been downloaded and only the placeholder file is present in EPIC_annotation/raw/

## Inputs

- MOFA2 latent-factor weight matrix for the buccal DNA methylation view
- Infinium MethylationEPIC v1.0 product annotation file (from illumina.com, replacing the placeholder in EPIC_annotation/raw/)
- EWAS atlas catalogue of CpG-trait associations
- List of top 100 CpG probe identifiers per methylation factor

## Outputs

- Factor x trait enrichment table with overlap-size and significance (FDR-corrected) columns, written to the Nextflow --output directory

## How to apply

Load the MOFA2 (R/Bioconductor MOFA2 1.3.4) weight matrix for the buccal DNA methylation view and rank CpG probes by the magnitude of their weights, retaining the top 100 CpGs per factor. Map the selected probe identifiers to traits using the Infinium MethylationEPIC product annotation (the placeholder file in `EPIC_annotation/raw/` must be replaced with the Illumina product file) together with the EWAS atlas catalogue of CpG-trait associations. Run over-representation testing (hypergeometric / Fisher's exact test) of each factor's CpG set against trait-specific CpG sets, recording overlap size and nominal p-value per factor-trait pair, then correct p-values with FDR across all factor-trait combinations. Assemble the results into a factor x trait enrichment table with overlap-size and significance columns and write it to the Nextflow --output directory.

## Related tools

- **MOFA2** (Supplies the latent-factor weight matrix from which the top 100 CpGs per methylation factor are ranked and selected)
- **EWAS atlas** (Reference catalogue of CpG-trait associations used to map probe identifiers and define trait-specific CpG sets for over-representation testing)
- **Nextflow** (Runs the factor-annotation module of the ACTIONdemonstrator pipeline (action.nf), writing the enrichment table to --output)

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv
```

## Evaluation signals

- The enrichment table file exists in the --output directory
- The table contains one entry per factor-trait pair
- Overlap-size and significance fields are populated for every factor-trait combination
- Enrichment p-values are FDR-corrected across all factor-trait combinations
- Methylation factors recover trait enrichments consistent with the reported glucocorticoid, aging, and immune associations

## Limitations

- The Infinium MethylationEPIC product file is not shipped with the repository for storage/licensing reasons; the user must download it separately from Illumina, so a fresh clone runs incomplete without it
- Privacy regulations prohibit sharing the real ACTION Biomarker Study methylation data; only synthetic example data are provided, so enrichment results on synthetic data are illustrative only
- Only the top 10% most variable probes among the 787,711 autosomal probes that survived QC entered upstream analyses, so trait recovery is limited to factors whose CpGs fall within this variable-probe subset

## Evidence

- [methods] download the missing Infinium MethylationEPIC product file: "download the missing Infinium MethylationEPIC product file"
- [readme] for which a placeholder is located in `EPIC_annotation/raw/`: "for which a placeholder is located in `EPIC_annotation/raw/`"
- [discussion] From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses.: "From the 787,711 autosomal methylation probes that survived QC, the top 10% most variable probes were included in the analyses."
- [results] the top 100 CpGs with the largest weights and performs enrichment analysis within the EWAS atlas: [evidence span withheld — anchor status: not_found]
