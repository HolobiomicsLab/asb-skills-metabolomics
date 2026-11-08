---
name: ko-gene-mapping
description: Use when the input is a gene-symbol or gene-ID abundance table (e.g.
  transcriptomics, scRNA-seq) that must be converted to a K number (K00001-style)
  KO abundance table before running ko.test() and reporter_score() in ReporterScore.
when_to_use_negative:
- The input is already a KO (K number), compound (C number), or microbial-taxonomy
  abundance table — mapping would be redundant; use the matching built-in database
  instead
- The abundance table has been prefiltered to a candidate list, since GRSA requires
  the full background feature set to be threshold-free
- Feature identifiers cannot be resolved to gene symbols resolvable by the organism
  pathway database (e.g. unknown species without a KEGG org entry)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ReporterScore
  role: Provides gene2ko(), load_org_pathway()/custom_modulelist_from_org(), ko.test()
    and reporter_score() for gene-to-KO mapping and subsequent GRSA enrichment
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Prerequisite dependency installed from GitHub before loading ReporterScore
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install the GitHub development versions of pcutils and ReporterScore
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:8011e27d67050796fd2e9d2b534221dcf92486d9cb8de30acc638ba67e7d548a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - GSA
  - GSVA
  - KO_enrich
  - KO_gsa
  - KO_gsea
  - KO_gsva
  - KO_padog
  - KO_safe
  - PADOG
  - R
  - ReporterScore
  - clusterProfiler
  - devtools
  - pcutils
  - safe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ko-gene-mapping

## Summary

Maps gene-level (or compound/microbe-level) abundance tables onto KEGG Orthology (KO) identifiers using the ReporterScore package's gene2ko() function and organism pathway databases, so that generalized reporter score-based enrichment analysis (GRSA) can be run on pathway/module KO-level inputs.

## When to use

Use when the input is a gene-symbol or gene-ID abundance table (e.g. transcriptomics, scRNA-seq) that must be converted to a K number (K00001-style) KO abundance table before running ko.test() and reporter_score() in ReporterScore.

## When NOT to use

- The input is already a KO (K number), compound (C number), or microbial-taxonomy abundance table — mapping would be redundant; use the matching built-in database instead
- The abundance table has been prefiltered to a candidate list, since GRSA requires the full background feature set to be threshold-free
- Feature identifiers cannot be resolved to gene symbols resolvable by the organism pathway database (e.g. unknown species without a KEGG org entry)

## Inputs

- Gene-level abundance table (rownames = gene symbols such as 'PEX11A'; colnames = samples; values = read counts or TPM/FPKM/RPKM/relative abundance)
- Sample metadata table (rownames = samples matching abundance colnames; columns = groups/time points)
- Organism pathway database from load_org_pathway() (e.g. 'hsa' or 'mmu') or a custom modulelist

## Outputs

- KO-level abundance table with rownames as KEGG K numbers (e.g. 'K03169')
- KO-level statistics from ko.test() feeding downstream reporter_score()/get_reporter_score() enrichment

## How to apply

Load the ReporterScore package in R (after installing pcutils via devtools::install_github("Asa12138/pcutils") and ReporterScore via devtools::install_github("Asa12138/ReporterScore")). Obtain the organism-specific pathway database with load_org_pathway() (built-in human 'hsa' and mouse 'mmu' pathway-ko/gene/compound databases, updatable by KEGG API with update_org_pathway()), then call gene2ko() to convert the gene abundance table rownames (gene symbols) into KEGG K numbers. The resulting KO abundance table keeps samples as columns and must keep its full background (no prefiltering), because GRSA is a threshold-free method. Pair the table with metadata whose rownames match the abundance-table colnames, then pass both to ko.test()/reporter_score() with the chosen statistical method. Decision point: if the data are already KO-, compound-, or taxonomy-level, skip the gene-to-KO step and use the appropriate built-in database (load_KOlist(), load_CPDlist(), custom_modulelist()).

## Related tools

- **ReporterScore** (Provides gene2ko(), load_org_pathway()/custom_modulelist_from_org(), ko.test() and reporter_score() for gene-to-KO mapping and subsequent GRSA enrichment) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Prerequisite dependency installed from GitHub before loading ReporterScore) — github.com/Asa12138/pcutils
- **devtools** (Used to install the GitHub development versions of pcutils and ReporterScore)

## Examples

```
library(ReporterScore); kodf <- gene2ko(gene_abundance, modulelist = load_org_pathway(org = "hsa")); res <- reporter_score(kodf, group = "Group", metadata = metadata)
```

## Evaluation signals

- Output KO table rownames all match the KEGG K-number format (e.g. K00001) and sample colnames are preserved from the input gene table
- rownames(metadata) match (or partially match) colnames of the KO abundance table; all(rownames(metadata) %in% colnames(KO_abundance)) returns TRUE
- The abundance table retains the full unfiltered background (no candidate-list prefiltering) as required by the threshold-free GRSA
- Downstream reporter_score()/get_reporter_score() returns reporter scores and adjusted P-values for pathways containing the mapped KOs
- Database version is checked/updated (built-in KEGG 2023-08 version; update_org_pathway()/update_KO_file() via KEGG API) so annotations are current

## Limitations

- Only organisms with available KEGG organism pathway databases (built-in hsa and mmu; others downloadable via KEGG API) can be mapped directly
- No version number is stated for the ReporterScore package or its bundled 2023-08 KEGG databases, limiting reproducibility unless update functions are run
- Compound IDs must be standardized to KEGG C numbers before enrichment; unstandardized compound IDs cannot be mapped

## Evidence

- [readme] The rownames are feature ids (e.g. “K00001” (KEGG K number) if feature=“ko”; “PEX11A” (gene symbol) if feature=“gene”: [evidence span withheld — anchor status: external_doc]
- [readme] the input abundance table should not be prefiltered: [evidence span withheld — anchor status: external_doc]
- [readme] ReporterScore automatically matches samples via rownames/colnames: [evidence span withheld — anchor status: external_doc]
- [other] gene2ko mapping grounded in task workflow: [evidence span withheld — anchor status: not_found]
- [methods] install commands for pcutils and ReporterScore: "devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore)"
