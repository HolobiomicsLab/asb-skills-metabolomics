---
name: metabolite-significance-testing
description: Use when you have metabolite names (or names + concentrations) with sample
  metadata from a Metabolomics Workbench study and need to determine which metabolites
  are significantly altered between two groups (e.g., Normal vs Cirrhosis) before
  enrichment analysis.
when_to_use_negative:
- The input is raw spectra or an unannotated feature table with no metabolite names
  and metadata — significant_met() expects MW-style metabolite/concentration tables
  retrievable via getmwstudies().
- The goal is already-computed pathway or metabolite-class enrichment scores for a
  curated metabolite list; use path_enrichmentscore()/metclassenrichment() instead
  of re-running significance testing.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetENP
  role: R package whose significant_met(), anova_ana(), and plot_volcano() functions
    implement the significance testing on MW study tables
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime environment in which the MetENP significance functions are sourced
    and executed
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:38086bb15a903ad4e4461ef81fb4ffd7562fb0d93a6ee3d03cd5b9fd4ecb460a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolite-significance-testing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Statistically identify significantly altered metabolites from a Metabolomics Workbench metabolite table (names plus concentrations with metadata) using MetENP's significant_met()/anova_ana() functions, applying a missing-value filter, p-value cutoff with p_adjust correction, and a log2 fold-change cutoff.

## When to use

Use when you have metabolite names (or names + concentrations) with sample metadata from a Metabolomics Workbench study and need to determine which metabolites are significantly altered between two groups (e.g., Normal vs Cirrhosis) before enrichment analysis.

## When NOT to use

- The input is raw spectra or an unannotated feature table with no metabolite names and metadata — significant_met() expects MW-style metabolite/concentration tables retrievable via getmwstudies().
- The goal is already-computed pathway or metabolite-class enrichment scores for a curated metabolite list; use path_enrichmentscore()/metclassenrichment() instead of re-running significance testing.

## Inputs

- Metabolite names table from getmwstudies(study_id, 'metabolites')
- Concentration/intensity table with sample metadata from getmwstudies(study_id, 'factors'/'metabolites')
- Two-group factor comparison specification (e.g., Normal vs Cirrhosis, prednisolone vs tetracosactide, ridA-mutant vs wild type)
- Filter parameters: NA-handling option (50percent or half_of_min), p-value cutoff, log2FC cutoff

## Outputs

- Table of significantly altered metabolites with p-values, adjusted p-values, and log2 fold-changes
- Volcano plot of altered metabolites (plot_volcano)
- Filtered significant-metabolite list passed onward to metclassenrichment() and met_pathways()

## How to apply

Fetch the study's metabolite and metadata tables via getmwstudies(studyid, typedata), then run significant_met() on the two-group comparison. Choose a missing-value (NA) handling filter appropriate to the study — the case studies used '50percent' (canine plasma ST001140, human liver ST000915) and 'half_of_min' (Salmonella ST001308). Apply a p-value cutoff of p<0.05 with p_adjust correction and a log2 fold-change cutoff of 0.5, as in all three published case studies. Standardize metabolite names with convert_refmet() before downstream mapping so significance results can be joined to KEGG compound IDs. The rationale is that enrichment scoring (metclassenrichment, path_enrichmentscore) should operate only on the statistically significant subset, not the full metabolite list.

## Related tools

- **MetENP** (R package whose significant_met(), anova_ana(), and plot_volcano() functions implement the significance testing on MW study tables) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime environment in which the MetENP significance functions are sourced and executed)

## Examples

```
source('path/significant_met.R')
```

## Evaluation signals

- ST000915 (Normal vs Cirrhosis, p<0.05 with p_adjust, log2FC cutoff 0.5, 50percent NA filter) must yield 85 significantly altered metabolites.
- ST001140 (prednisolone vs tetracosactide, 50percent filter, p 0.05, log2FC 0.5) must yield 26 significantly altered metabolites.
- ST001308 (ridA-mutant vs wild type, half_of_min NA handling, p_adjust 0.05) must yield 5 significant metabolites spanning Amino acids and Fatty acids subclasses.
- Significant-metabolite count after filtering must equal the input count minus metabolites failing the NA filter; no metabolite above the p_adjust or log2FC cutoff should remain in the output.
- Volcano plot (plot_volcano) should show up- and down-regulated metabolites consistent with the log2FC cutoff.

## Limitations

- Cutoffs (p<0.05, log2FC 0.5) and NA-handling choices are user-specified; no statistical validation metrics or quantitative benchmarks are reported for the three case studies, so default cutoffs are inherited from the published examples rather than benchmarked.
- Requires metabolite names that can be standardized via RefMet (convert_refmet); unstandardizable names will fail downstream KEGG mapping.
- No software version number or changelog is stated for MetENP, so results may vary across package versions — follow the GitHub page to ensure the latest version.

## Evidence
