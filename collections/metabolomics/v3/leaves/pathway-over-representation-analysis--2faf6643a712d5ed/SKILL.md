---
name: pathway-over-representation-analysis
description: Use when you have a curated list of dysregulated metabolites (e.g., Log2FC
  > 1.5, BH-adj p < 0.05) from LC–MS metabolomics and want to identify over-represented
  biological pathways such as lipid or amino-acid metabolism.
when_to_use_negative:
- The metabolite list is from raw, uncorrected data and may be too sparse or contaminated
  with batch drift — the article noted the raw-data list contained too few metabolites
  from these pathways to trigger enrichment.
- You need pathway-level abundance changes rather than presence/absence over-representation;
  this method tests whether a pre-defined pathway's compounds are over-represented,
  not whether pathway activity changed.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0091
tools:
- name: ConsensusPathDB
  role: Web tool performing over-representation analysis of the metabolite list against
    the KEGG pathways database using hypergeometric test with FDR correction
- name: HMDB
  role: Database used to search and filter the metabolite list to retain only potential
    metabolite hits before pathway enrichment
  repo: http://www.hmdb.ca
- name: KEGG pathways database
  role: Predefined compound pathway reference (4289 compound IDs) against which over-representation
    is computed
- name: dbnorm
  role: R package used to produce the batch-effect-corrected metabolite intensity
    table from which the differential metabolite list is derived
  repo: https://github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:8274b01194da6702555f7695eb9237180c89fd423cadd8b0bb4ad6035d6c63b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusPathDB
  - HMDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - dbnorm
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathway-over-representation-analysis

## Summary

Over-representation (MSEA) analysis of a differential metabolite list against the KEGG compound pathway database using ConsensusPathDB's hypergeometric test with FDR correction, to identify biological pathways enriched among dysregulated metabolites (e.g., alpha-Linolenic acid metabolism, fatty acid biosynthesis).

## When to use

Use when you have a curated list of dysregulated metabolites (e.g., Log2FC > 1.5, BH-adj p < 0.05) from LC–MS metabolomics and want to identify over-represented biological pathways such as lipid or amino-acid metabolism.

## When NOT to use

- The metabolite list is from raw, uncorrected data and may be too sparse or contaminated with batch drift — the article noted the raw-data list contained too few metabolites from these pathways to trigger enrichment.
- You need pathway-level abundance changes rather than presence/absence over-representation; this method tests whether a pre-defined pathway's compounds are over-represented, not whether pathway activity changed.

## Inputs

- List of differential metabolites (e.g., from a volcano-plot analysis with Log2FC > 1.5 and BH-adjusted p < 0.05)
- HMDB metabolite identifiers
- Batch-effect-corrected metabolite intensity table (e.g., ber-corrected LC–MS data)

## Outputs

- Table of significantly over-represented KEGG pathways (≥2 metabolites, q-value < 0.05)
- Per-pathway metabolite coverage (relative number of input metabolites defining each pathway relative to the whole pathway metabolite set)
- Negative log2-transformed q-values for pathway significance
- CSV file of pathway enrichment results

## How to apply

Take the differential metabolite list (thresholds such as Log2FC > 1.5 and BH-adjusted p-value < 0.05 as in the HFD analysis) and first filter it against the Human Metabolome Database (HMDB, http://www.hmdb.ca) to retain only potential metabolite hits. Submit the HMDB-confirmed metabolite identifiers to the ConsensusPathDB web interface (http://consensuspathdb.org/) and select over-representation analysis against the predefined KEGG pathways database (4289 compound IDs). ConsensusPathDB computes pathway significance with the hypergeometric test and returns q-values after false discovery rate correction. Accept pathways meeting a minimum of two metabolites per pathway and q-value < 0.05; report metabolite coverage (relative number of input metabolites defining each pathway relative to the whole pathway metabolite set) alongside negative log2-transformed q-values.

## Related tools

- **ConsensusPathDB** (Web tool performing over-representation analysis of the metabolite list against the KEGG pathways database using hypergeometric test with FDR correction)
- **HMDB** (Database used to search and filter the metabolite list to retain only potential metabolite hits before pathway enrichment) — http://www.hmdb.ca
- **KEGG pathways database** (Predefined compound pathway reference (4289 compound IDs) against which over-representation is computed)
- **dbnorm** (R package used to produce the batch-effect-corrected metabolite intensity table from which the differential metabolite list is derived) — https://github.com/NBDZ/dbnorm

## Examples

```
Submit HMDB-filtered metabolite list at http://consensuspathdb.org/ with KEGG pathways (4289 compound IDs); retain pathways with ≥2 metabolites and q-value < 0.05.
```

## Evaluation signals

- Enriched pathways must include lipid metabolism categories such as alpha-Linolenic acid metabolism and fatty acid biosynthesis, each represented by ≥2 metabolites with q-value < 0.05
- All retained metabolites must be HMDB-confirmed hits after filtering against HMDB
- Every reported pathway must have a coverage value and a negative log2-transformed q-value
- The number of metabolites defining each enriched pathway must be ≥2
- FDR-corrected q-values must be < 0.05 for every reported pathway

## Limitations

- Batch effect correction must be applied first: the raw-data metabolite list contained too few metabolites from the lipid pathways to trigger enrichment, so pathway signals can be lost without proper correction
- Only pathways with at least two metabolites in the input list are reported, so sparsely populated pathways are invisible
- Over-representation is limited to the KEGG compound IDs predefined in ConsensusPathDB (4289 compound IDs)

## Evidence

- [results] MSEA reveals lipid metabolism pathways enriched only with the batch-effect-corrected list: [evidence span withheld — anchor status: not_found]
- [results] Thresholds used to define the significant metabolite list: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [methods] Filtering step against HMDB before pathway analysis: "searched and filtered against a Human Metabolome Database (HMDB; http://www.hmdb.ca)"
- [methods] KEGG database size and mapping target: "The list of identifiers was mapped to predefined KEGG pathways database enlisted by 4289 compound IDs."
- [methods] ConsensusPathDB hypergeometric test with FDR correction and ≥2 metabolites criterion: [evidence span withheld — anchor status: not_found]
- [methods] Coverage definition: [evidence span withheld — anchor status: not_found]
