---
name: pubmed-literature-factor-interpretation
description: Use when a MOFA (or, in BiomiX ≥2.5, DIABLO) integration has produced
  latent factors and you need to biologically interpret discriminant factors by linking
  their top contributing features to published literature, clinical variables, and
  pathways.
when_to_use_negative:
- When the analysis goal is single-omics differential expression (DESeq2/Limma, ChAMP,
  TidyMass) rather than interpreting latent MOFA factors
- When no discriminant factors have been identified and factor tuning is still needed
  rather than factor interpretation
- When only correlation-based annotation is required without literature or pathway
  context (e.g. X-omics ACTION-style correlation-only workflows)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: BiomiX
  role: Runs MOFA integration and the factor-interpretation module combining clinical
    correlation, EnrichR/GSEA pathway analysis, and automated PubMed bibliography
    search
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the latent factors and their top contributors that the PubMed-based
    interpretation annotates
  repo: https://github.com/IxI-97/BiomiX
- name: EnrichR
  role: Pathway/biological-process exploration of top contributing features alongside
    the literature annotation
- name: GSEA
  role: Biological process analysis of discriminant factor contributors complementing
    PubMed hits
- name: PubMed
  role: Source of the automated bibliography search returning the most related articles
    per discriminant factor contributor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:a39c2ac2aafcb55d4bce669936b7984f428c73e7d3ab03db6bcf3743f53463c5
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
  - ChAMP
  - DESeq2
  - EnrichR
  - GSEA
  - Limma
  - MOFA (Multi-Omics Factor Analysis)
  - PubMed
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pubmed-literature-factor-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates discriminant MOFA factors produced by BiomiX using an automated PubMed bibliography search that returns the articles most related to each factor's top contributors, complemented by clinical-data correlation and EnrichR/GSEA pathway exploration.

## When to use

Use when a MOFA (or, in BiomiX ≥2.5, DIABLO) integration has produced latent factors and you need to biologically interpret discriminant factors by linking their top contributing features to published literature, clinical variables, and pathways.

## When NOT to use

- When the analysis goal is single-omics differential expression (DESeq2/Limma, ChAMP, TidyMass) rather than interpreting latent MOFA factors
- When no discriminant factors have been identified and factor tuning is still needed rather than factor interpretation
- When only correlation-based annotation is required without literature or pathway context (e.g. X-omics ACTION-style correlation-only workflows)

## Inputs

- Fitted MOFA model (e.g. ten-factor model) with discriminant factors identified in BiomiX
- List of top contributing features per discriminant factor across transcriptomics, methylomics, metabolomics, or undefined omics
- Clinical annotation metadata (e.g. IGHV mutation status, 11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13)
- BiomiX factor-interpretation module outputs from EnrichR/GSEA biological process analysis

## Outputs

- Per-factor annotation table (e.g. mofa_factor_annotation_cll.csv) with clinical correlations, EnrichR/GSEA pathway terms, and PubMed literature hits per top contributor
- Most-related PubMed articles for each discriminant factor contributor
- Factor-clinical variable correlation results

## How to apply

After running MOFA integration in BiomiX with factor-number tuning (e.g. the ten-factor CLL model), identify the factors that discriminate cohorts (e.g. factors 1, 3, 4, 5 and 6 separating mutated vs unmutated IGHV CLL patients). In the factor-interpretation module, correlate each discriminant factor with clinical annotations (e.g. 11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13), explore top contributing pathways with EnrichR and GSEA biological process analysis, and run the automated PubMed bibliography search that returns the most related articles per top contributor. Compile the clinical correlations, pathway terms, and literature hits per factor into an annotation table such as mofa_factor_annotation_cll.csv. BiomiX is notable as the first tool to exploit PubMed bibliography to annotate hidden factors, and it annotates factors via three methods (correlation, pathway analysis, bibliography) unlike tools that only perform correlation analysis. Undefined omics contributors are also recognized by the bibliography and correlation analysis following the version 2.4 bug fix.

## Related tools

- **BiomiX** (Runs MOFA integration and the factor-interpretation module combining clinical correlation, EnrichR/GSEA pathway analysis, and automated PubMed bibliography search) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the latent factors and their top contributors that the PubMed-based interpretation annotates) — https://github.com/IxI-97/BiomiX
- **EnrichR** (Pathway/biological-process exploration of top contributing features alongside the literature annotation)
- **GSEA** (Biological process analysis of discriminant factor contributors complementing PubMed hits)
- **PubMed** (Source of the automated bibliography search returning the most related articles per discriminant factor contributor)

## Evaluation signals

- The annotation table (e.g. mofa_factor_annotation_cll.csv) records, for each discriminant factor, clinical correlations, EnrichR/GSEA pathway terms, and PubMed literature hits
- The MOFA model contains the expected number of factors, of which the expected subset discriminates the cohorts (e.g. ten factors with five discriminating mutated vs unmutated IGHV CLL)
- Bibliography and correlation outputs correctly include contributors from undefined omics (post version 2.4 bug fix behavior)
- Each factor's interpretation is supported by all three annotation methods (correlation, pathway analysis, bibliography), not correlation alone

## Limitations

- BiomiX supports transcriptomics, metabolomics, methylomics and undefined omics; proteomics and genomics are not yet supported
- Users are required to carefully review datasets before use; there is no automated validation of inputs
- Earlier versions had a bug where bibliography and correlation analysis did not recognize undefined omics contributors (fixed in version 2.4)

## Evidence
