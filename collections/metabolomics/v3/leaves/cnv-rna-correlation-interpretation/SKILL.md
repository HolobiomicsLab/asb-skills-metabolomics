---
name: cnv-rna-correlation-interpretation
description: Use when a user asks what GutUDB reports for one gene's CNV frequency
  and RNA expression in an intestinal disease (e.g. METTL3 in colorectal cancer) and
  whether the two are correlated.
when_to_use_negative:
- The request concerns a gene absent from GutUDB's 260,790 disease-gene associations
  or outside the 56 covered intestinal diseases — the database has no entry to interpret.
- The task requires raw CNV segmentation files or expression count matrices; GutUDB's
  Browse/Query interface exposes curated summaries, not raw omics files.
- The question is about methylation or m6A (the 1908 miCLIP-seq sites) rather than
  CNV–RNA concordance — a different module of the database is appropriate.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3295
tools:
- name: GutUDB
  role: Web-based database whose Browse/Query functionality retrieves the gene's CNV
    deletion frequency and RNA expression annotation across the genomics and transcriptomics
    layers
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:56037613adbf22e55df8a5eb061fb59b527ef765fa6b4f791208193f53e644aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cnv-rna-correlation-interpretation

## Summary

Interprets GutUDB copy-number-variation and transcriptomics layers jointly for a single gene, pairing quantified CNV frequencies with annotated RNA expression status to characterize the gene's multi-omics behavior in intestinal disease cohorts.

## When to use

Use when a user asks what GutUDB reports for one gene's CNV frequency and RNA expression in an intestinal disease (e.g. METTL3 in colorectal cancer) and whether the two are correlated.

## When NOT to use

- The request concerns a gene absent from GutUDB's 260,790 disease-gene associations or outside the 56 covered intestinal diseases — the database has no entry to interpret.
- The task requires raw CNV segmentation files or expression count matrices; GutUDB's Browse/Query interface exposes curated summaries, not raw omics files.
- The question is about methylation or m6A (the 1908 miCLIP-seq sites) rather than CNV–RNA concordance — a different module of the database is appropriate.

## Inputs

- GutUDB query result for the target gene in the genomics layer (CNV entry, e.g. deletion frequency 0.04 for METTL3 in colorectal cancer)
- GutUDB query result for the same gene in the transcriptomics layer (RNA expression annotation, e.g. 'significantly high RNA expression')
- Article text containing the qualitative CNV–RNA correlation conclusion (e.g. 'weak correlation')

## Outputs

- Structured JSON record (e.g. mettl3_cnv_rna_observations.json) containing exactly three gene-level observations: CNV deletion frequency, RNA expression status, and the CNV–RNA correlation statement
- Validated annotation set confirming each observation matches the GutUDB-reported value verbatim

## How to apply

First pull the gene's entry from GutUDB's genomics layer, which aggregates 54,131 CNVs, to obtain the disease-specific CNV deletion frequency (e.g. METTL3 deletion frequency 0.04 in colorectal cancer). Next retrieve the matching transcriptomics annotation — GutUDB holds 137 RNA-seq data sets with 7.9 million transcriptional profiles — and record the qualitative expression call (here 'significantly high RNA expression'). Then assemble the two values plus the paper's stated correlation direction ('weak correlation between CNV and RNA expression') into a single structured record such as mettl3_cnv_rna_observations.json. Validate the record by checking that exactly three observations — the 0.04 deletion frequency, the high-RNA annotation, and the weak-correlation statement — are present and match the GutUDB annotation values verbatim. The rationale is that GutUDB stores CNV and RNA evidence in separate omics layers, so a correct joint interpretation must cite values from each layer rather than inferring one from the other.

## Related tools

- **GutUDB** (Web-based database whose Browse/Query functionality retrieves the gene's CNV deletion frequency and RNA expression annotation across the genomics and transcriptomics layers) — https://github.com/Ansanqi/GutUDB

## Examples

```
query GutUDB with gene='METTL3', disease='colorectal cancer', omics=['CNV','RNA-seq'] and assemble the returned deletion frequency, expression annotation, and correlation note into mettl3_cnv_rna_observations.json
```

## Evaluation signals

- Output JSON contains exactly the three expected observations: CNV deletion frequency 0.04, 'significantly high RNA expression' annotation, and the weak CNV–RNA correlation statement.
- Each quantified value in the JSON matches the number reported in the GutUDB annotation/article verbatim (0.04, not a rounded or recomputed variant).
- The correlation statement in the output mirrors the article's own wording ('weak correlation between CNV and RNA expression') rather than a paraphrase.
- No extra fields, genes, or disease terms beyond the single target gene and disease appear in the JSON record.
- Record is valid JSON and named consistently with the workflow's specified artifact (mettl3_cnv_rna_observations.json).

## Limitations

- Correlation statements in GutUDB annotations are qualitative summaries from the source publication, not recomputed statistics — users should not treat them as new quantitative analyses.
- GutUDB covers 56 intestinal diseases across six species; results for genes or diseases outside this scope cannot be retrieved.
- CNV calls (54,131 entries) and RNA profiles (7.9 million entries) are stored in separate omics layers, so any joint interpretation is only as consistent as the original curation of both layers.

## Evidence

- [abstract] GutUDB currently includes 72,248 single-nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes"
- [abstract] 137 RNA-seq data sets related to 7.9 million transcriptional profiles: "137 RNA‐seq data sets related to 7.9 million transcriptional profiles"
- [abstract] four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
