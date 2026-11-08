---
name: pathway-network-construction
description: Use when a metabolomics feature list must be mapped to pathway database
  identifiers (KEGG compound IDs) to enable pathway-based network construction or
  enrichment, and when you need to quantify how many features are actually annotatable.
when_to_use_negative:
- The dataset contains many exogenous, secondary-metabolism, or unknown metabolites
  and the goal is unbiased network analysis — use a data-driven approach such as the
  DNEA R package (GLASSO partial correlation networks) instead
- The metabolite list is already a set of KEGG compound IDs requiring no name-to-ID
  mapping step
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3407
tools:
- name: FELLA
  role: Pathway-database-dependent tool that requires KEGG compound IDs to build its
    network from KEGG pathway definitions; used to expose the pathway-coverage gap
- name: R
  role: Programming environment in which the ID mapping, network construction, and
    coverage report are performed
- name: DNEA R package
  role: Source of the TEDDY metabolomics example data and the data-driven alternative
    that does not require KEGG annotation
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:08ee7a551fe715db6c97813418412f285264c416712213b0f7bfd7c03b10714d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - FELLA
  - R
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pathway-network-construction

## Summary

Construction of pathway-database-dependent biological networks requires mapping metabolite features to reference identifiers (e.g., KEGG compound IDs) before tools like FELLA can build networks from pathway definitions; this skill covers the ID-mapping step and quantifying the resulting pathway-coverage gap.

## When to use

Use when a metabolomics feature list must be mapped to pathway database identifiers (KEGG compound IDs) to enable pathway-based network construction or enrichment, and when you need to quantify how many features are actually annotatable.

## When NOT to use

- The dataset contains many exogenous, secondary-metabolism, or unknown metabolites and the goal is unbiased network analysis — use a data-driven approach such as the DNEA R package (GLASSO partial correlation networks) instead
- The metabolite list is already a set of KEGG compound IDs requiring no name-to-ID mapping step

## Inputs

- Metabolomics feature table with metabolite names (e.g., TEDDY metabolomics data, Metabolomics Workbench ST001386; 174 metabolites)
- KEGG pathway/compound definitions used to construct the pathway network

## Outputs

- Per-metabolite annotation table recording whether each metabolite maps to a KEGG compound ID and enters the analysis
- Summary coverage counts (e.g., 19 of 174 metabolites annotated) written to a report such as fella_coverage_report.csv
- Pathway network built from KEGG definitions over the mapped metabolites only

## How to apply

Load the metabolite feature names from the metabolomics dataset (e.g., the 174 metabolites in the TEDDY dataset from Metabolomics Workbench ST001386, also shipped with the DNEA R package). Attempt to map each metabolite name to a KEGG compound ID, since pathway-network tools such as FELLA require KEGG identifiers for network construction. Build the network from KEGG pathway definitions using only the successfully mapped metabolites, and record for each input feature whether it is annotated and therefore enters the analysis. Compute the summary coverage counts — in the TEDDY/FELLA case, only 19 of 174 metabolites were KEGG-annotated and included — to expose the coverage gap of pathway-based enrichment relative to data-driven alternatives like DNEA, which can handle exogenous, secondary, and unknown metabolites.

## Related tools

- **FELLA** (Pathway-database-dependent tool that requires KEGG compound IDs to build its network from KEGG pathway definitions; used to expose the pathway-coverage gap)
- **R** (Programming environment in which the ID mapping, network construction, and coverage report are performed)
- **DNEA R package** (Source of the TEDDY metabolomics example data and the data-driven alternative that does not require KEGG annotation) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
BiocManager::install("DNEA"); metabolites <- featureNames(TEDDY_data); kegg_ids <- mapIds(orgMetab, metabolites, 'KEGG', 'NAME'); coverage <- sum(!is.na(kegg_ids))
```

## Evaluation signals

- The per-metabolite table must contain one row per input metabolite (174 for TEDDY) with a binary KEGG-annotation flag
- The summary counts must reproduce 19 out of 174 TEDDY metabolites annotated in KEGG and included in the FELLA analysis
- The constructed pathway network must contain only metabolites with valid KEGG compound IDs; unmapped features must be absent from the network node set
- The coverage report (fella_coverage_report.csv) must include both the per-metabolite annotation table and the aggregate counts

## Limitations

- FELLA requires KEGG IDs, limiting the analysis to metabolites annotated in KEGG — only 19 of 174 TEDDY metabolites were included
- Pathway databases contain limited information about lipid and secondary metabolic pathways, excluding many measured compounds
- Pathway analysis cannot map unknown or exogenous metabolites to a human pathway

## Evidence

- [discussion] FELLA requires KEGG IDs, limiting the analysis to the metabolites that are annotated in KEGG. As a result, only 19 out of 174 metabolites present in the TEDDY data were included in the analysis.: "FELLA requires KEGG IDs, limiting the analysis to the metabolites that are annotated in KEGG. As a result, only 19 out of 174 metabolites present in the TEDDY data were included in the analysis."
- [intro] Pathway analysis works well for endogenous metabolites that can be mapped to metabolic pathways (e.g. primary metabolites). However, the coverage of lipid and secondary metabolism by pathway: "Pathway analysis works well for endogenous metabolites that can be mapped to metabolic pathways (e.g. primary metabolites). However, the coverage of lipid and secondary metabolism by pathway"
- [abstract] To our knowledge, this package is the only publicly available tool designed for the construction of biological networks and subsequent enrichment testing for datasets containing exogenous, secondary,: "To our knowledge, this package is the only publicly available tool designed for the construction of biological networks and subsequent enrichment testing for datasets containing exogenous, secondary,"
- [readme] Example data is also stored in the package and instructions on how to access it is available in the vignette.: [evidence span withheld — anchor status: external_doc]
