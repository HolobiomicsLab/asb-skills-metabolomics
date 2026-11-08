---
name: multi-omics-factor-literature-annotation
description: Use when a discriminant MOFA/DIABLO factor from BiomiX needs biological
  interpretation and you want literature-grounded annotation of its multi-omics contributor
  features via PubMed abstracts.
when_to_use_negative:
- The factor is not discriminant (not separating cohorts/conditions) — BiomiX's bibliography
  search is designed for discriminative factor interpretation
- No known contributor feature names exist for the factor, so no keyword lists can
  be compiled for the three-level PubMed search
- Interpretation via clinical-data correlation or pathway analysis (EnrichR/GSEA)
  alone is sufficient and literature mining is unnecessary
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3360
tools:
- name: BiomiX
  role: Provides the bibliography research module and MOFA/DIABLO contributor outputs
    feeding the PubMed search
  repo: https://github.com/IxI-97/BiomiX
- name: PubMed
  role: Source of abstracts retrieved by the three-level contributor keyword search
- name: litsearchR
  role: Text-mining of retrieved abstracts to extract and reduce candidate keywords
    to non-redundant 2–4-word combinations
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the discriminant factors whose contributors drive the literature
    queries
- name: R
  role: Language in which the three-level PubMed search and keyword filtering are
    implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:324677c8a33153402b0713c2a6f2ece226fc9464632e1597a3c8464715aaaeb8
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
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-omics-factor-literature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates discriminant MOFA (or DIABLO) factor contributors by mining PubMed: a three-level contributor-keyword literature search retrieves abstracts, then litsearchR text mining reduces abstract-derived keywords to non-redundant 2–4-word terms mapped to GO-BP and HPO vocabularies for biological interpretation.

## When to use

Use when a discriminant MOFA/DIABLO factor from BiomiX needs biological interpretation and you want literature-grounded annotation of its multi-omics contributor features via PubMed abstracts.

## When NOT to use

- The factor is not discriminant (not separating cohorts/conditions) — BiomiX's bibliography search is designed for discriminative factor interpretation
- No known contributor feature names exist for the factor, so no keyword lists can be compiled for the three-level PubMed search
- Interpretation via clinical-data correlation or pathway analysis (EnrichR/GSEA) alone is sufficient and literature mining is unnecessary

## Inputs

- Discriminant MOFA (or DIABLO) factor contributor list from BiomiX interpretation output (feature names per omics modality)
- BiomiX multi-omics integration output (transcriptomics, metabolomics, methylomics contributions)
- PubMed bibliographic database (query endpoint)
- Gene Ontology biological process and HPO vocabulary reference terms

## Outputs

- pubmed_factor_bibliography.tsv with columns PubMedID, DOI, per-level match counts, and matched keywords
- Non-redundant 2–4-word keyword list mapped to GO-BP / HPO terms for factor annotation

## How to apply

Compile contributor keyword lists (feature names per omics modality) for the discriminant factor from the BiomiX MOFA interpretation output, then construct a three-level PubMed search in R: level 1 queries all omics contributor terms together, level 2 pairwise omics combinations, and level 3 each single omics separately — the multi-omics levels are prioritized so merged contributors rank above single-omics hits. Count contributor matches per retrieved article and retain only abstracts matching at least 1/10 (≥1 of 10) of the factor's contributors. Assemble results into a .tsv with PubMedID, DOI, per-level match counts, and matched keywords (pubmed_factor_bibliography.tsv). Finally run the litsearchR R package on the retrieved abstracts to extract candidate keywords, reduce them to non-redundant 2–4-word combinations, and filter against Gene Ontology biological process and HPO vocabularies to keep only biologically interpretable annotations. Note BiomiX 2.5 extends the same interpretation pipeline (clinical, pathway, bibliography) to DIABLO factors, and fixed recognition of 'undefined' omics contributors in the bibliography step.

## Related tools

- **BiomiX** (Provides the bibliography research module and MOFA/DIABLO contributor outputs feeding the PubMed search) — https://github.com/IxI-97/BiomiX
- **PubMed** (Source of abstracts retrieved by the three-level contributor keyword search)
- **litsearchR** (Text-mining of retrieved abstracts to extract and reduce candidate keywords to non-redundant 2–4-word combinations)
- **MOFA (Multi-Omics Factor Analysis)** (Produces the discriminant factors whose contributors drive the literature queries)
- **R** (Language in which the three-level PubMed search and keyword filtering are implemented)

## Examples

```
Rscript run_biomiX_bibliography.R --factor discriminant_factor_1 --omics transcriptomics,metabolomics,methylomics --out pubmed_factor_bibliography.tsv
```

## Evaluation signals

- Every row of pubmed_factor_bibliography.tsv contains a resolvable PubMedID and a resolvable DOI
- Every row records at least one contributor match count and at least one matched keyword
- Each retained abstract matches at least 1/10 of the discriminant factor's contributors
- The filtered keyword list contains only 2–4-word terms mapped to GO biological process or HPO vocabulary terms
- LitsearchR output keywords are non-redundant after reduction

## Limitations

- Only abstracts matching ≥1 of 10 contributors are retained, so factors with few or poorly named contributors may yield sparse bibliographies
- BiomiX is R/GUI-based and 'not a definitive solution'; users must carefully review datasets before use and there is no automated validation
- Bug history: bibliography and correlation analysis initially failed to recognize undefined-omics contributors; fixed in BiomiX 2.4; Mac installation still unstable — prefer Linux/Windows or the planned Nextflow workflow

## Evidence

- [abstract] three-level PubMed search prioritizing merged contributors: [evidence span withheld — anchor status: not_found]
- [abstract] 1/10 contributor match threshold: [evidence span withheld — anchor status: not_found]
- [readme] README: DIABLO bibliography pipeline extension: [evidence span withheld — anchor status: not_found]
