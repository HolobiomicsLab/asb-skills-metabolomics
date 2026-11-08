---
name: litsearchr-keyword-extraction
description: 'Use when annotating a discriminant MOFA (or DIABLO) factor: once PubMed
  abstracts have been retrieved for the factor''s top omics contributors, litsearchr
  text-mining converts those abstracts into a biologically interpretable keyword list.'
when_to_use_negative:
- Factor interpretation already uses the clinical-correlation or pathway-analysis
  branches (EnrichR/GSEA) and no literature annotation is requested.
- No discriminant factor contributors are available (e.g. the integration output has
  no significant factors), so the three-level PubMed query cannot be built.
- The user's goal is general literature search rather than interpreting a specific
  multi-omics factor — the pipeline is tied to factor contributor keywords.
tools:
- name: litsearchr
  role: R package used for the text-mining step that extracts candidate keywords from
    retrieved PubMed abstracts and reduces them to non-redundant combinations
- name: BiomiX
  role: 'Hosts the bibliography research module: builds the three-level PubMed queries
    over factor contributors and writes the pubmed_factor_bibliography.tsv output'
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Runtime environment in which the BiomiX bibliography module and litsearchr
    keyword extraction are executed
- name: PubMed
  role: Source of abstracts most related to the discriminant factor contributors,
    queried at three levels and used for keyword extraction
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the discriminant factors and their top multi-omics contributors that
    seed the PubMed keyword queries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:e86a822d363de76d3429f08fae6b570fbfc2d8b7d457c4ab27e7c9cb5f240171
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - MOFA (Multi-Omics Factor Analysis)
  - PubMed
  - R
  - litsearchr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# litsearchr-keyword-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts candidate keywords from PubMed abstracts retrieved around a discriminant MOFA factor's multi-omics contributors using the litsearchr R package, then reduces them to non-redundant 2-4 word terms and keeps only Gene Ontology biological process or HPO-mapped annotations. It is the BiomiX literature-annotation layer that turns factor contributors into interpretable biological concepts.

## When to use

Use when annotating a discriminant MOFA (or DIABLO) factor: once PubMed abstracts have been retrieved for the factor's top omics contributors, litsearchr text-mining converts those abstracts into a biologically interpretable keyword list.

## When NOT to use

- Factor interpretation already uses the clinical-correlation or pathway-analysis branches (EnrichR/GSEA) and no literature annotation is requested.
- No discriminant factor contributors are available (e.g. the integration output has no significant factors), so the three-level PubMed query cannot be built.
- The user's goal is general literature search rather than interpreting a specific multi-omics factor — the pipeline is tied to factor contributor keywords.

## Inputs

- BiomiX MOFA interpretation output providing top contributor feature names per omics modality for the discriminant factor
- PubMed abstracts retrieved by the three-level contributor query
- Gene Ontology biological process and HPO vocabularies for keyword filtering

## Outputs

- pubmed_factor_bibliography.tsv with columns PubMedID, DOI, per-level match counts, and matched keywords
- filtered list of non-redundant two-to-four-word keyword terms mapped to GO-BP or HPO terms

## How to apply

First compile the contributor keyword lists (feature names per omics modality) for the discriminant MOFA factor from the BiomiX MOFA interpretation output, and build the three-level PubMed search (level 1: all omics contributor terms together; level 2: pairwise omics combinations; level 3: each single omics separately). Count contributor matches per retrieved article and retain only articles matching at least 1 out of 10 of the factor's contributors, assembling PubMedID, DOI, per-level match counts and matched keywords into pubmed_factor_bibliography.tsv. Then run litsearchr text mining on the retrieved abstracts to extract candidate keywords and reduce them to non-redundant two-to-four-word combinations. Finally filter the extracted keyword combinations against Gene Ontology biological process and HPO vocabulary terms so only biologically interpretable annotations survive. The rationale is to integrate literature knowledge into MOFA factor interpretation, since BiomiX 'implemented, for the first time, the factor identification through bibliography research on Pubmed'.

## Related tools

- **litsearchr** (R package used for the text-mining step that extracts candidate keywords from retrieved PubMed abstracts and reduces them to non-redundant combinations)
- **BiomiX** (Hosts the bibliography research module: builds the three-level PubMed queries over factor contributors and writes the pubmed_factor_bibliography.tsv output) — https://github.com/IxI-97/BiomiX
- **R** (Runtime environment in which the BiomiX bibliography module and litsearchr keyword extraction are executed)
- **PubMed** (Source of abstracts most related to the discriminant factor contributors, queried at three levels and used for keyword extraction)
- **MOFA (Multi-Omics Factor Analysis)** (Produces the discriminant factors and their top multi-omics contributors that seed the PubMed keyword queries)

## Evaluation signals

- Every row of pubmed_factor_bibliography.tsv contains a resolvable PubMedID and a resolvable DOI.
- Each output row records at least one contributor match count and at least one matched keyword.
- The retained articles match at least 1 out of 10 of the factor's contributors in the abstract text.
- The filtered keyword list contains only two-to-four-word terms that map to Gene Ontology biological process or HPO terms (no redundant terms remain).

## Limitations

- BiomiX's bibliography annotation is tied to its factor-interpretation pipeline and the currently supported omics modules; the tool supports transcriptomics, metabolomics and methylomics but proteomics and genomics are not yet implemented.
- In BiomiX 2.3/2.4 the bibliography and correlation analysis originally failed for 'Undefined' omics contributors; this was fixed (bug fixed so they 'are now properly recognizing the undefined omics contributors'), but users should verify their BiomiX version.
- Users must carefully review input datasets before use — there is no automated validation of inputs feeding the bibliography module.
- BiomiX is described by its authors as a first attempt, not a definitive solution; the keyword-extraction thresholds (e.g. the 1/10 contributor match rule) are tool defaults rather than established community standards.

## Evidence
