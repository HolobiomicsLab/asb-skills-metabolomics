---
name: mofa-contributor-keyword-matching
description: 'Use when interpreting a discriminant MOFA factor from a BiomiX multi-omics
  run: you have per-omics contributor (feature) keyword lists and want literature-backed,
  ontology-mapped biological annotations of the factor.'
when_to_use_negative:
- The goal is factor annotation by clinical data correlation or pathway analysis (EnrichR/GSEA)
  — BiomiX uses three distinct annotation methods, and this skill covers only the
  PubMed bibliography route
- The MOFA factor is not discriminant (does not distinguish cohorts), since the module
  is built to interpret contributors of discriminant factors
- Contributor feature names are not usable as text keywords (e.g. unnormalized identifiers);
  note that BiomiX 2.5 added Ensembl name conversion in MOFA contributors output to
  address this
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3068
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3957
tools:
- name: BiomiX
  role: R package providing the PubMed bibliography research module that queries,
    filters, and tabulates literature matches for MOFA factor contributors
  repo: https://github.com/IxI-97/BiomiX
- name: PubMed
  role: Literature database queried at three levels (all-omics, pairwise-omics, single-omics)
    for contributor terms
- name: litsearchR
  role: R package used to text-mine retrieved abstracts and reduce keywords to non-redundant
    two-to-four-word combinations
- name: MOFA (Multi-Omics Factor Analysis)
  role: Supplies the discriminant factor and its per-omics contributors that drive
    the PubMed queries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:bbc0f8b0131cd4e8ff61eaf1cea58396c6f99784504ecf23334986acb8b8af3a
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
  - litsearchR
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3068
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mofa-contributor-keyword-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BiomiX's PubMed bibliography module annotates discriminant MOFA factors by running three-level PubMed searches over contributor keyword lists from each input omics, retaining abstracts matching at least 1/10 of the factor's contributors, then extracting non-redundant keywords with litsearchr and mapping them to GO-BP/HPO terms.

## When to use

Use when interpreting a discriminant MOFA factor from a BiomiX multi-omics run: you have per-omics contributor (feature) keyword lists and want literature-backed, ontology-mapped biological annotations of the factor.

## When NOT to use

- The goal is factor annotation by clinical data correlation or pathway analysis (EnrichR/GSEA) — BiomiX uses three distinct annotation methods, and this skill covers only the PubMed bibliography route
- The MOFA factor is not discriminant (does not distinguish cohorts), since the module is built to interpret contributors of discriminant factors
- Contributor feature names are not usable as text keywords (e.g. unnormalized identifiers); note that BiomiX 2.5 added Ensembl name conversion in MOFA contributors output to address this

## Inputs

- Contributor keyword lists (feature names per omics modality) for a discriminant MOFA factor, from the BiomiX MOFA interpretation output
- PubMed abstracts retrieved by the three-level query
- Gene Ontology biological process and HPO vocabularies for keyword filtering

## Outputs

- pubmed_factor_bibliography.tsv with columns PubMedID, DOI, per-level match counts, and matched keywords
- Non-redundant two-to-four-word keyword list from litsearchr, filtered to GO-BP- or HPO-mapped terms annotating the MOFA factor

## How to apply

Compile the contributor keyword lists (feature names per omics modality) for the discriminant MOFA factor from the BiomiX MOFA interpretation output. In R, construct and run the three-level PubMed search: level 1 queries all omics contributor terms together, level 2 queries pairwise omics combinations, and level 3 queries each single omics separately — the prioritization of merged multi-omics contributors is the module's core design. Count contributor matches per retrieved article and retain only articles matching at least 1/10 of the factor's contributors. Assemble results into pubmed_factor_bibliography.tsv with PubMedID, DOI, per-level match counts, and matched keywords. Finally, run litsearchr text-mining on the retrieved abstracts to extract candidate keywords, reduce them to non-redundant two-to-four-word combinations, and filter against Gene Ontology biological process and HPO vocabularies to keep biologically interpretable annotations.

## Related tools

- **BiomiX** (R package providing the PubMed bibliography research module that queries, filters, and tabulates literature matches for MOFA factor contributors) — https://github.com/IxI-97/BiomiX
- **PubMed** (Literature database queried at three levels (all-omics, pairwise-omics, single-omics) for contributor terms)
- **litsearchR** (R package used to text-mine retrieved abstracts and reduce keywords to non-redundant two-to-four-word combinations)
- **MOFA (Multi-Omics Factor Analysis)** (Supplies the discriminant factor and its per-omics contributors that drive the PubMed queries)

## Examples

```
Rscript run_biomix_bibliography.R --factor factor_3 --omics transcriptomics,metabolomics,methylomics --output pubmed_factor_bibliography.tsv
```

## Evaluation signals

- Every row of pubmed_factor_bibliography.tsv contains a resolvable PubMedID and DOI
- Each retained row has at least one recorded contributor match count and at least one matched keyword
- Every retained article matches at least 1/10 of the factor's contributors (threshold check on match counts)
- The final keyword list contains only GO-BP- or HPO-mapped terms
- All extracted keywords are two-to-four-word non-redundant combinations after litsearchr reduction

## Limitations

- Abstracts are only accepted if at least 1 of 10 contributors (1/10 of the factor's contributors) appears in the text, which can miss relevant literature below this match threshold
- Bug in BiomiX ≤2.4: bibliography and correlation analysis did not recognize undefined-omics contributors; fixed in 2.4/2.5 notes, so undefined omics may be dropped in older versions
- BiomiX does not yet support proteomics and genomics, so contributors from those omics cannot feed this module
- Users must carefully review input datasets before use; there is no automated validation of inputs
- Not a standalone general literature-search method: it is designed specifically to annotate hidden/discriminant MOFA factors, the first tool to exploit a PubMed bibliography for this purpose

## Evidence

- [results] Retention threshold of at least 1/10 contributors per article: [evidence span withheld — anchor status: not_found]
- [readme] Ensembl name conversion improves keyword usability: [evidence span withheld — anchor status: not_found]
