---
name: ontology-vocabulary-keyword-filtering
description: Use when a literature-mining step has produced candidate keyword n-grams
  from abstracts and you need to reduce them to biologically interpretable terms by
  mapping against GO biological process and HPO vocabularies.
when_to_use_negative:
- The goal is pathway over-representation testing on a feature list rather than filtering
  text-mined abstract keywords — use EnrichR/GSEA instead, which BiomiX uses for biological
  process exploration.
- The input is already a curated gene/phenotype list mapped to ontologies; re-filtering
  n-grams adds nothing in that case.
- No ontology (GO-BP or HPO) coverage exists for the domain of the extracted keywords,
  so mapping-based filtering would discard everything.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: BiomiX
  role: Runs the PubMed bibliography research on discriminant MOFA factor contributors
    and assembles the keyword-matched bibliography table
  repo: https://github.com/IxI-97/BiomiX
- name: litsearchr
  role: R text-mining of retrieved abstracts to extract candidate keywords and reduce
    them to non-redundant two-to-four-word combinations
- name: PubMed
  role: Source of the abstracts whose keywords are extracted and ontology-filtered
- name: MOFA (Multi-Omics Factor Analysis)
  role: Provides the discriminant factors whose top contributors define the keyword
    lists driving the literature search
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:2343866981164f5e7c6863f7b4b045e75d80b145b2bc05f72f36a5f6d7a6c973
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# Ontology-vocabulary keyword filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters text-mined, non-redundant keyword combinations extracted from PubMed abstracts (via litsearchr) so that only terms mappable to Gene Ontology biological process (GO-BP) or Human Phenotype Ontology (HPO) vocabulary are kept, yielding biologically interpretable annotations of discriminant MOFA factor contributors in BiomiX.

## When to use

Use when a literature-mining step has produced candidate keyword n-grams from abstracts and you need to reduce them to biologically interpretable terms by mapping against GO biological process and HPO vocabularies.

## When NOT to use

- The goal is pathway over-representation testing on a feature list rather than filtering text-mined abstract keywords — use EnrichR/GSEA instead, which BiomiX uses for biological process exploration.
- The input is already a curated gene/phenotype list mapped to ontologies; re-filtering n-grams adds nothing in that case.
- No ontology (GO-BP or HPO) coverage exists for the domain of the extracted keywords, so mapping-based filtering would discard everything.

## Inputs

- Retrieved PubMed abstracts associated with discriminant MOFA factor contributors (articles matching at least 1/10 of the factor's contributors)
- Contributor keyword lists (feature names per omics modality) from the BiomiX MOFA interpretation output
- Gene Ontology biological process vocabulary terms
- Human Phenotype Ontology (HPO) vocabulary terms

## Outputs

- Filtered list of non-redundant two-to-four-word keyword combinations, each mapped to a GO-BP or HPO term
- pubmed_factor_bibliography.tsv with columns PubMedID, DOI, per-level match counts, and matched keywords

## How to apply

First run the litsearchr text-mining step on the abstracts retrieved by BiomiX's three-level PubMed search (all-omics contributors, pairwise omics combinations, single omics) to extract candidate keywords and reduce them to non-redundant two-to-four-word combinations. Then filter these combinations against Gene Ontology biological process and HPO vocabulary terms, keeping only keyword combinations that map to GO-BP or HPO terms so the resulting annotations are biologically interpretable. Rationale: BiomiX implements factor identification through PubMed bibliography research to integrate literature knowledge into MOFA factor interpretation, and ontology mapping prevents raw text n-grams from becoming unusable annotations. Validation per the source workflow: the filtered keyword list should contain only GO-BP- or HPO-mapped two-to-four-word terms, and the accompanying bibliography table rows must carry a resolvable PubMedID and DOI, at least one contributor match count (abstracts matching at least 1/10 of the factor's contributors are retained), and at least one matched keyword.

## Related tools

- **BiomiX** (Runs the PubMed bibliography research on discriminant MOFA factor contributors and assembles the keyword-matched bibliography table) — https://github.com/IxI-97/BiomiX
- **litsearchr** (R text-mining of retrieved abstracts to extract candidate keywords and reduce them to non-redundant two-to-four-word combinations)
- **PubMed** (Source of the abstracts whose keywords are extracted and ontology-filtered)
- **MOFA (Multi-Omics Factor Analysis)** (Provides the discriminant factors whose top contributors define the keyword lists driving the literature search)

## Evaluation signals

- Every keyword in the filtered list is a two-to-four-word combination and maps to a GO-BP or HPO vocabulary term.
- Every row of pubmed_factor_bibliography.tsv contains a resolvable PubMedID and DOI.
- Every retained article has at least one recorded contributor match count (threshold: at least 1/10 of the factor's contributors matched in the abstract).
- Every row has at least one matched keyword.
- The keyword list is non-redundant (litsearchr reduction applied) rather than containing duplicate or near-duplicate phrases.

## Limitations

- BiomiX is described as a first attempt, not a definitive solution; the bibliography module requires users to carefully review datasets before use, with no automated validation.
- Earlier versions failed to recognize contributors of undefined/undetermined omics in the bibliography and correlation analysis; this was only fixed in version 2.4, so older versions may silently miss those contributors.
- Proteomics and genomics analysers are not yet supported, constraining which omics contributor lists can drive the PubMed search.

## Evidence
