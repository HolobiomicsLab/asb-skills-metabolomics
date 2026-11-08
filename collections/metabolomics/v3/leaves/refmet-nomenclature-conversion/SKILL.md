---
name: refmet-nomenclature-conversion
description: Use when metabolite feature names from a Metabolomics Workbench study
  (or custom dataset) are unstandardized and must be normalized via RefMet before
  mapping to KEGG compound IDs and computing enrichment.
when_to_use_negative:
- Input metabolite names are already RefMet-standardized — re-running conversion adds
  an unnecessary network call and may fail to match
- Working with a non-metabolomic dataset (e.g., gene expression) where RefMet nomenclature
  does not apply
- Analysis goal is pathway enrichment without identifier standardization, using significant_met_own
  on custom pre-annotated data
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MetENP
  role: R package providing convert_refmet, which maps metabolite names to RefMet
    standardized nomenclature within the enrichment workflow
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: RefMet
  role: Reference list of metabolite names supplying the standard nomenclature that
    convert_refmet maps metabolite names onto
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:1c24dfe9a6daf65c44d5b0bba06f2235d1cfa36267ba29a9c94e9e3c6a258d82
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# refmet-nomenclature-conversion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardize metabolite names to RefMet nomenclature within the MetENP workflow before KEGG identifier mapping, so downstream enrichment and pathway analysis operate on consistent metabolite identities.

## When to use

Use when metabolite feature names from a Metabolomics Workbench study (or custom dataset) are unstandardized and must be normalized via RefMet before mapping to KEGG compound IDs and computing enrichment.

## When NOT to use

- Input metabolite names are already RefMet-standardized — re-running conversion adds an unnecessary network call and may fail to match
- Working with a non-metabolomic dataset (e.g., gene expression) where RefMet nomenclature does not apply
- Analysis goal is pathway enrichment without identifier standardization, using significant_met_own on custom pre-annotated data

## Inputs

- Metabolite intensity table (names or names and concentrations) retrieved via getmwstudies from a Metabolomics Workbench study such as ST001308
- Metabolite name column in original (unstandardized) nomenclature

## Outputs

- Data frame of metabolite names converted to RefMet standard nomenclature, ready for map_keggid KEGG compound mapping

## How to apply

After fetching metabolite intensities and metadata with getmwstudies(studyid, typedata), run convert_refmet on the metabolite name column to map each name to the RefMet standard nomenclature. Rationale: RefMet provides 'standard nomenclature for metabolites, enabling efficient comparisons between metabolite data across experiments', and MetENP's pipeline requires standardized classes before calling map_keggid (with the organism, e.g., Salmonella enterica) and metclassenrichment. Use convert_refmet either via the installed MetENP package or by sourcing 'path/convert_refmet.R' as in the README fallback. Validate that metabolite names were converted before proceeding to significant_met, since failed conversions will drop compounds from KEGG pathway association.

## Related tools

- **MetENP** (R package providing convert_refmet, which maps metabolite names to RefMet standardized nomenclature within the enrichment workflow) — https://github.com/metabolomicsworkbench/MetENP
- **RefMet** (Reference list of metabolite names supplying the standard nomenclature that convert_refmet maps metabolite names onto)

## Examples

```
source('path/convert_refmet.R'); data_refmet <- convert_refmet(metabolite_data)
```

## Evaluation signals

- Every input metabolite name in the study table returns a non-empty RefMet standard name (failed/blank mappings indicate unmatched compounds)
- Downstream map_keggid on the converted names yields KEGG compound IDs for the target organism (e.g., Salmonella enterica for ST001308)
- Metabolite classes assigned in metclassenrichment use RefMet-standardized categories (e.g., Amino acids, Fatty acids for the ST001308 case study)
- Comparing unconverted vs converted name columns shows no loss of metabolite count — each row retains a mapped identity

## Limitations

- Unrecognized metabolite names may not convert to RefMet nomenclature and will be lost in subsequent KEGG mapping steps
- No software version stated for MetENP; README notes 'Please follow the details at the GitHub page (this page) to ensure you use the latest version', so behavior of convert_refmet may change between versions

## Evidence

- [results] Workflow order: fetch data, standardize names, map KEGG IDs: [evidence span withheld — anchor status: not_found]
