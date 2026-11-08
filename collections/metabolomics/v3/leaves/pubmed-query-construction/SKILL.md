---
name: pubmed-query-construction
description: Use when a discriminant MOFA factor from a multi-omics integration has
  contributor feature lists (genes, metabolites, CpG probes) and you need literature
  evidence to interpret the factor via PubMed abstracts.
when_to_use_negative:
- Factor interpretation by clinical data correlation is sufficient and no literature
  annotation is required
- Input factor contributors are not resolvable as searchable text keywords (e.g. unnamed
  spectral features without annotation)
- You need pathway enrichment rather than literature annotation — use the EnrichR/GSEA
  module instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3063
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3572
tools:
- name: BiomiX
  role: Encapsulates the three-level PubMed search and bibliography module for MOFA
    factor interpretation
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Language in which the PubMed bibliography research module and keyword extraction
    are implemented
- name: PubMed
  role: Bibliographic database queried for articles most related to the discriminant
    factor contributors
- name: litsearchr
  role: R package used for text-mining retrieved abstracts to extract candidate keywords
    reduced to non-redundant two-to-four-word combinations
- name: MOFA (Multi-Omics Factor Analysis)
  role: Source of the discriminant factor and its contributor lists that seed the
    PubMed queries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:b2d9f13cd5e0a4804560be32a6ec0f091d75dd03b35629e60d45a55ece778391
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
  edam_topics:
  - http://edamontology.org/topic_3063
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pubmed-query-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs three-level PubMed bibliographic queries from the contributor keyword lists of a discriminant MOFA factor to retrieve and filter literature that annotates the factor's multi-omics contributors. This is BiomiX's literature-based factor interpretation step.

## When to use

Use when a discriminant MOFA factor from a multi-omics integration has contributor feature lists (genes, metabolites, CpG probes) and you need literature evidence to interpret the factor via PubMed abstracts.

## When NOT to use

- Factor interpretation by clinical data correlation is sufficient and no literature annotation is required
- Input factor contributors are not resolvable as searchable text keywords (e.g. unnamed spectral features without annotation)
- You need pathway enrichment rather than literature annotation — use the EnrichR/GSEA module instead

## Inputs

- Contributor keyword lists per omics modality for a discriminant MOFA factor (from BiomiX MOFA interpretation output)
- PubMed as the bibliographic search backend (via R, e.g. rentrez/easyPubMed style queries)

## Outputs

- pubmed_factor_bibliography.tsv containing PubMedID, DOI, per-level match counts, and matched keywords per retrieved article
- Filtered keyword list of biologically interpretable two-to-four-word terms mapped to GO biological process or HPO vocabulary

## How to apply

Compile the contributor keyword lists (feature names per omics modality) for the discriminant MOFA factor from the BiomiX MOFA interpretation output. Construct the three-level PubMed search in R: level 1 queries all omics contributor terms together, level 2 queries pairwise omics combinations, and level 3 queries each single omics separately — this prioritizes abstracts merging multi-omics contributors. Count contributor matches per retrieved article and retain only articles matching at least 1/10 of the factor's contributors. Assemble results into a .tsv table (PubMedID, DOI, per-level match counts, matched keywords), then run the litsearchR package text-mining step on the retrieved abstracts to extract non-redundant two-to-four-word keyword combinations, filtered against Gene Ontology biological process and HPO vocabulary terms.

## Related tools

- **BiomiX** (Encapsulates the three-level PubMed search and bibliography module for MOFA factor interpretation) — https://github.com/IxI-97/BiomiX
- **R** (Language in which the PubMed bibliography research module and keyword extraction are implemented)
- **PubMed** (Bibliographic database queried for articles most related to the discriminant factor contributors)
- **litsearchr** (R package used for text-mining retrieved abstracts to extract candidate keywords reduced to non-redundant two-to-four-word combinations)
- **MOFA (Multi-Omics Factor Analysis)** (Source of the discriminant factor and its contributor lists that seed the PubMed queries)

## Examples

```
Rscript BiomiX_pubmed_bibliography.R --factor factor_3 --omics transcriptomics,metabolomics --min_contributors 1 --out pubmed_factor_bibliography.tsv
```

## Evaluation signals

- Every row of pubmed_factor_bibliography.tsv contains a resolvable PubMedID and a resolvable DOI
- Every row has at least one recorded contributor match count across the three search levels
- Every row has at least one matched keyword from the factor's contributor list
- Retained articles each match at least 1 out of 10 (≥1/10) of the factor's contributors in the abstract text
- The filtered keyword list contains only two-to-four-word combinations mapped to GO biological process or HPO terms

## Limitations

- Only abstracts matching at least one out of ten contributors are retained, so factors with few well-known contributors may yield sparse bibliography results
- BiomiX (v2.x) has limited omics analysers; proteomics and genomics were not yet supported, constraining which contributor types can seed the queries
- Users must carefully review datasets before use; there is no automated validation of contributor lists feeding the search

## Evidence

- [abstract] Three-level PubMed search prioritizing merged contributors: [evidence span withheld — anchor status: not_found]
- [other] litsearchr keyword extraction and vocabulary filtering: [evidence span withheld — anchor status: not_found]
