---
name: clinical-variable-factor-correlation
description: Use when a fitted MOFA (or DIABLO) model contains discriminant factors
  separating cohorts and you need to interpret those factors by correlating them with
  clinical/metadata variables and pathway/literature annotations.
when_to_use_negative:
- The multiomics data has not yet been integrated (no MOFA/DIABLO model fitted) —
  run the integration step first
- The analysis goal is single-omics differential analysis (DESeq2/Limma, ChAMP, TidyMass)
  rather than interpreting latent multiomics factors
- No clinical or metadata annotations are available to correlate against the factors
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: BiomiX
  role: Provides the factor-interpretation module that correlates discriminant MOFA/DIABLO
    factors with clinical data and runs pathway and PubMed annotation
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the latent factors and discriminant factor contributors that are
    correlated with clinical variables
- name: EnrichR
  role: Biological process exploration of the top contributing pathways of discriminant
    factors
- name: GSEA
  role: Biological process exploration of the top contributing pathways of discriminant
    factors
- name: PubMed
  role: Automated bibliography search returning the articles most related to discriminant
    factor contributors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:d7630c0b53f61898e83b30a12831fc658fcf09e153d688404a33a430d1b8ce30
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# clinical-variable-factor-correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correlates discriminant MOFA factors (e.g., the five IGHV-discriminating factors in the CLL multiomics case study) with clinical annotations such as 11q22.3 deletion, trisomy 12, gender, and TP53/SF3B1/del17p13 status using BiomiX's factor-interpretation module, to assign biological/clinical meaning to latent factors.

## When to use

Use when a fitted MOFA (or DIABLO) model contains discriminant factors separating cohorts and you need to interpret those factors by correlating them with clinical/metadata variables and pathway/literature annotations.

## When NOT to use

- The multiomics data has not yet been integrated (no MOFA/DIABLO model fitted) — run the integration step first
- The analysis goal is single-omics differential analysis (DESeq2/Limma, ChAMP, TidyMass) rather than interpreting latent multiomics factors
- No clinical or metadata annotations are available to correlate against the factors

## Inputs

- Fitted MOFA model with tuned number of factors (e.g., ten factors) and identified discriminant factors
- Clinical/metadata annotation table with variables such as 11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13 status
- MOFA factor top-contributor feature lists per omics (transcriptomics, methylomics, metabolomics)

## Outputs

- Per-factor clinical correlation results linking discriminant factors to clinical variables
- EnrichR and GSEA biological process pathway terms for top contributing features per factor
- PubMed literature hits for top factor contributors
- Compiled factor annotation table (e.g., mofa_factor_annotation_cll.csv)

## How to apply

After running MOFA integration in BiomiX and tuning the factor number (e.g., a ten-factor model), identify the factors that discriminate the cohorts of interest (e.g., factors 1, 3, 4, 5 and 6 separating mutated vs unmutated IGHV CLL patients). Use BiomiX's factor-interpretation module to correlate each discriminant factor with the clinical annotation columns supplied in the metadata (e.g., 11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13). Complement the correlations by exploring top contributing pathways with EnrichR and GSEA biological process analysis, and annotate top factor contributors with BiomiX's automated PubMed bibliography search. Compile per-factor clinical correlations, pathway terms, and literature hits into an annotation table such as mofa_factor_annotation_cll.csv. BiomiX version 2.5 extends this same interpretation pipeline (clinical, pathway analysis, bibliography) to DIABLO factors.

## Related tools

- **BiomiX** (Provides the factor-interpretation module that correlates discriminant MOFA/DIABLO factors with clinical data and runs pathway and PubMed annotation) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the latent factors and discriminant factor contributors that are correlated with clinical variables)
- **EnrichR** (Biological process exploration of the top contributing pathways of discriminant factors)
- **GSEA** (Biological process exploration of the top contributing pathways of discriminant factors)
- **PubMed** (Automated bibliography search returning the articles most related to discriminant factor contributors)

## Evaluation signals

- The correlation output must include each named clinical variable (11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13) for each discriminant factor
- The MOFA model must contain the tuned number of factors (ten) of which the expected subset (five) discriminate the IGHV cohorts
- The compiled annotation table must record, for each discriminant factor, both EnrichR/GSEA pathway terms and PubMed literature hits
- BiomiX v2.4+ bug check: bibliography and correlation analysis must properly recognize undefined-omics contributors, not only transcriptomics/methylomics/metabolomics
- All discriminant factors (not only significant ones, per BiomiX 2.5) should have contribution plots/heatmaps available for interpretation

## Limitations

- BiomiX relies on three methods (clinical correlation, pathway analysis, PubMed bibliography) to annotate factors, whereas comparable tools like X-omics ACTION only perform correlation analysis — but coverage depends on literature and metadata quality
- Users are required to carefully review datasets before use; BiomiX performs no automated validation of inputs
- MOFA can manage missing data during integration, but imputation choices (NIPALS, random forest, lasso, mean/median, 0 values) may bias correlation results

## Evidence

- [readme] BiomiX 2.5 embeds DIABLO into the same clinical/pathway/bibliography interpretation pipeline as MOFA: [evidence span withheld — anchor status: not_found]
