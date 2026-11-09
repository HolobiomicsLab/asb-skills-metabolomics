---
name: hypergeometric-enrichment-testing
description: Use when a list of significant metabolites (e.g. Log2FC > 1.5, BH-adjusted
  p < 0.05) must be mapped to KEGG pathways to identify over-represented biological
  processes after batch correction.
when_to_use_negative:
- When the input is a full metabolite abundance matrix rather than a curated list
  of significant hits — MSEA here requires an already-filtered differential list.
- When the goal is gene set enrichment (e.g. GO or Reactome gene sets); this workflow
  is specific to metabolite identifiers (HMDB/KEGG compound IDs) and KEGG pathway
  over-representation.
- When the metabolite list has not yet been batch-effect corrected (e.g. via dbnorm
  ber/ComBat), as raw lists contained too few metabolites from lipid pathways to trigger
  enrichment.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_0154
tools:
- name: ConsensusPathDB
  role: Web tool that performs the hypergeometric over-representation analysis of
    HMDB-filtered metabolite identifiers against KEGG pathways and returns q-value–corrected
    enrichment results.
- name: KEGG pathways database
  role: Reference pathway collection (4289 compound IDs) against which metabolite
    set over-representation is tested.
- name: Human Metabolome Database (HMDB)
  role: Used to search and filter the significant metabolite list, retaining only
    potential metabolite hits before enrichment submission.
- name: dbnorm
  role: Upstream batch-effect correction tool (e.g. ber model) whose corrected metabolite
    list is the input for enrichment analysis; without this correction lipid pathways
    may not reach the ≥2-metabolite, q < 0.05 threshold.
  repo: github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:666ae441d22be915c1df015399ed7bf47a23c67ed3b1cd043ecb61d0def7c0b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusPathDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - dbnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hypergeometric-enrichment-testing

## Summary

Downstream metabolite set enrichment analysis (MSEA) via ConsensusPathDB, applying a hypergeometric over-representation test against KEGG pathways with FDR (q-value) correction, as used to identify lipid metabolism pathways enriched among HFD-dysregulated metabolites.

## When to use

Use when a list of significant metabolites (e.g. Log2FC > 1.5, BH-adjusted p < 0.05) must be mapped to KEGG pathways to identify over-represented biological processes after batch correction.

## When NOT to use

- When the input is a full metabolite abundance matrix rather than a curated list of significant hits — MSEA here requires an already-filtered differential list.
- When the goal is gene set enrichment (e.g. GO or Reactome gene sets); this workflow is specific to metabolite identifiers (HMDB/KEGG compound IDs) and KEGG pathway over-representation.
- When the metabolite list has not yet been batch-effect corrected (e.g. via dbnorm ber/ComBat), as raw lists contained too few metabolites from lipid pathways to trigger enrichment.

## Inputs

- Differential metabolite list (e.g. CSV) with Log2FC > 1.5 and BH-adjusted p < 0.05
- Metabolite identifiers filterable against HMDB (Human Metabolome Database)
- KEGG compound IDs mapped to KEGG pathways database (4289 compound IDs)

## Outputs

- Pathway enrichment table (CSV) with metabolite coverage and negative log2-transformed q-values
- Significantly dysregulated KEGG pathways (≥2 metabolites, q-value < 0.05)

## How to apply

Filter the differential metabolite list (Log2FC > 1.5, BH-adjusted p < 0.05) against the Human Metabolome Database (HMDB) to retain only potential metabolite hits, then submit HMDB identifiers to ConsensusPathDB for over-representation analysis against the predefined KEGG pathways database (4289 compound IDs). ConsensusPathDB computes pathway significance with a hypergeometric test and corrects p-values for multiple testing using false discovery rate q-values. Select significantly dysregulated pathways meeting the criteria of at least two metabolites per pathway and q-value < 0.05. Save the pathway enrichment table, including metabolite coverage (relative number of input metabolites defining each pathway relative to the whole pathway metabolite set) and negative log2-transformed q-values, as a CSV file. This step was applied after dbnorm batch-effect correction (e.g. ber) of the SKIPOGH/HFD LC–MS data, so the input list should come from batch-corrected differential analysis.

## Related tools

- **ConsensusPathDB** (Web tool that performs the hypergeometric over-representation analysis of HMDB-filtered metabolite identifiers against KEGG pathways and returns q-value–corrected enrichment results.)
- **KEGG pathways database** (Reference pathway collection (4289 compound IDs) against which metabolite set over-representation is tested.)
- **Human Metabolome Database (HMDB)** (Used to search and filter the significant metabolite list, retaining only potential metabolite hits before enrichment submission.)
- **dbnorm** (Upstream batch-effect correction tool (e.g. ber model) whose corrected metabolite list is the input for enrichment analysis; without this correction lipid pathways may not reach the ≥2-metabolite, q < 0.05 threshold.) — github.com/NBDZ/dbnorm

## Examples

```
Submit HMDB-filtered metabolite IDs to ConsensusPathDB over-representation analysis (http://consensuspathdb.org/) against KEGG pathways (4289 compound IDs); export the resulting enrichment table (metabolite coverage + −log2(q-value)) as CSV.
```

## Evaluation signals

- The enrichment output must include lipid metabolism pathways — specifically alpha-Linolenic acid metabolism and fatty acid biosynthesis — each represented by at least 2 metabolites with q-value < 0.05.
- Every submitted identifier must survive HMDB filtering; identifiers not found in HMDB should be excluded before submission to ConsensusPathDB.
- Q-values must be FDR-corrected (q-value) rather than raw hypergeometric p-values; verify the CSV contains both metabolite coverage and negative log2-transformed q-values.
- The enrichment must be performed on the batch-corrected (e.g. ber) differential list; the raw-data list should not produce the same lipid pathway enrichment (raw list contained too few metabolites from these pathways to trigger enrichment).
- Output CSV must contain a per-pathway 'metabolite coverage' value defined as the relative number of input metabolites defining each pathway relative to the whole pathway metabolite set.

## Limitations

- Only KEGG pathways with at least two input metabolites and q-value < 0.05 are reported, so pathways with single-metabolite coverage or marginal significance are silently dropped.
- The hypergeometric test depends on the predefined KEGG compound mapping (4289 compound IDs); metabolites not mappable to KEGG cannot contribute to pathway enrichment.
- Because the analysis was downstream of dbnorm batch correction, results are sensitive to the choice of correction model — raw or poorly corrected lists (e.g. lowess) may fail to reproduce the lipid pathway enrichment.

## Evidence

- [methods] The list of identifiers was mapped to predefined KEGG pathways database enlisted by 4289 compound IDs.: "The list of identifiers was mapped to predefined KEGG pathways database enlisted by 4289 compound IDs."
- [methods] pathway significance was computed with the hypergeometric test: [evidence span withheld — anchor status: not_found]
- [methods] significantly dysregulated pathways, based on the criteria of at least two metabolites per pathway and q-value < 0.05: [evidence span withheld — anchor status: not_found]
- [results] Relative number of metabolites defining each pathway relative to the whole pathway metabolite set: [evidence span withheld — anchor status: not_found]
- [results] raw-data list contained too few metabolites from these pathways to trigger enrichment: [evidence span withheld — anchor status: not_found]
