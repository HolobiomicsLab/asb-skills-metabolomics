---
name: cll-ighv-cohort-discriminant-factor-analysis
description: Use when a multi-omics cohort (transcriptomics, methylomics, metabolomics)
  carries binary condition labels such as mutated vs unmutated IGHV, and the goal
  is to fine-tune a MOFA model to find and interpret the factors that discriminate
  the two patient groups.
when_to_use_negative:
- Input omics types outside BiomiX's supported set — proteomics and genomics are not
  yet supported, so a proteomics-only or genomics-only dataset is the wrong fit
- Analysis goal is clustering without labeled cohorts — BiomiX's MOFA factor-tuning
  and discriminant-factor interpretation is designed for cohort-distinguishing models
  with condition labels
- Data has not been reviewed by the user — BiomiX strongly recommends manual dataset
  review before use and performs no automated validation, so blindly feeding uncurated
  data is inappropriate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: R-based GUI orchestrating the Preview-QC interface, single-omics pipelines,
    MOFA tuning and the factor-interpretation module (clinical correlation, EnrichR/GSEA,
    PubMed)
  repo: https://github.com/IxI-97/BiomiX
- name: BiomiX-consortium/BiomiX2.5
  role: Newer BiomiX release hosting version 2.5 with DIABLO/NEMO/SNF integration
    and bug fixes relevant to factor interpretation
  repo: https://github.com/BiomiX-consortium/BiomiX2.5
- name: MOFA (Multi-Omics Factor Analysis)
  role: Multi-omics integration engine whose factor number is tuned to ten; identifies
    shared sources of variation and the factors discriminating IGHV cohorts
- name: DESeq2
  role: Transcriptomics differential expression analysis feeding the integration
- name: Limma
  role: Transcriptomics differential expression analysis alongside DESeq2
- name: ChAMP
  role: Methylomics analysis pipeline within BiomiX
- name: TidyMass
  role: Untargeted metabolomics peak annotation and Wilcoxon testing with FDR correction;
    fragmentation spectra also used with CEU Mass Mediator for m/z annotation
- name: EnrichR
  role: Biological process exploration of top contributing pathways of discriminant
    factors
- name: GSEA
  role: Biological process exploration alongside EnrichR for discriminant factor contributors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:40a83fa162ccfb6a2e03dfa19190f0ceaa5b82fccde670fcf78bfd85254cdfbb
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
  - BiomiX-consortium/BiomiX2.5
  - ChAMP
  - DESeq2
  - EnrichR
  - GSEA
  - Limma
  - MOFA (Multi-Omics Factor Analysis)
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cll-ighv-cohort-discriminant-factor-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the BiomiX multi-omics integration workflow on the CLL transcriptomics–methylomics–metabolomics dataset to tune a ten-factor MOFA model, identify the five factors that discriminate mutated vs unmutated IGHV patients, and annotate those factors via clinical correlation, EnrichR/GSEA pathways, and PubMed literature mining.

## When to use

Use when a multi-omics cohort (transcriptomics, methylomics, metabolomics) carries binary condition labels such as mutated vs unmutated IGHV, and the goal is to fine-tune a MOFA model to find and interpret the factors that discriminate the two patient groups.

## When NOT to use

- Input omics types outside BiomiX's supported set — proteomics and genomics are not yet supported, so a proteomics-only or genomics-only dataset is the wrong fit
- Analysis goal is clustering without labeled cohorts — BiomiX's MOFA factor-tuning and discriminant-factor interpretation is designed for cohort-distinguishing models with condition labels
- Data has not been reviewed by the user — BiomiX strongly recommends manual dataset review before use and performs no automated validation, so blindly feeding uncurated data is inappropriate

## Inputs

- CLL multiomics dataset from http://pace.embl.de/ covering transcriptomics, methylomics and metabolomics with IGHV mutation status as condition labels
- Clinical annotation table with variables such as 11q22.3 deletion, trisomy 12, gender, TP53, SF3B1 and del17p13
- Expression/methylation/metabolite count tables plus metadata compatible with the BiomiX Shiny interfaces

## Outputs

- mofa_factor_annotation_cll.csv with clinical correlations, EnrichR/GSEA pathway terms and PubMed literature hits per discriminant factor
- Ten-factor MOFA model with five identified discriminant factors (1, 3, 4, 5, 6) separating mutated vs unmutated IGHV patients
- Single-omics differential results (DESeq2/Limma, ChAMP, TidyMass Wilcoxon with FDR) feeding the MOFA integration

## How to apply

Load the CLL multi-omics matrix with IGHV mutation status as CONDITION labels into the BiomiX R-based GUI and use the Preview-QC Shiny interface (PCA, UMAP, correlation heatmaps) to remove outliers based on their distances from PCA centroids and drop highly variable features likely driven by noise. Run the single-omics pipelines: DESeq2/Limma for transcriptomics, ChAMP for methylomics, and TidyMass-based untargeted metabolomics with Wilcoxon testing and False Discovery Rate correction. Run MOFA integration with factor-number tuning set to a ten-factor model and identify which factors separate mutated from unmutated IGHV cohorts (expected: factors 1, 3, 4, 5 and 6). Correlate the discriminant factors with clinical annotations (11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13) in the factor-interpretation module, then explore top contributing pathways with EnrichR and GSEA biological process analysis. Finally, run BiomiX's automated PubMed bibliography search to return the most related articles per top contributor and compile everything into mofa_factor_annotation_cll.csv.

## Related tools

- **BiomiX** (R-based GUI orchestrating the Preview-QC interface, single-omics pipelines, MOFA tuning and the factor-interpretation module (clinical correlation, EnrichR/GSEA, PubMed)) — https://github.com/IxI-97/BiomiX
- **BiomiX-consortium/BiomiX2.5** (Newer BiomiX release hosting version 2.5 with DIABLO/NEMO/SNF integration and bug fixes relevant to factor interpretation) — https://github.com/BiomiX-consortium/BiomiX2.5
- **MOFA (Multi-Omics Factor Analysis)** (Multi-omics integration engine whose factor number is tuned to ten; identifies shared sources of variation and the factors discriminating IGHV cohorts)
- **DESeq2** (Transcriptomics differential expression analysis feeding the integration)
- **Limma** (Transcriptomics differential expression analysis alongside DESeq2)
- **ChAMP** (Methylomics analysis pipeline within BiomiX)
- **TidyMass** (Untargeted metabolomics peak annotation and Wilcoxon testing with FDR correction; fragmentation spectra also used with CEU Mass Mediator for m/z annotation)
- **EnrichR** (Biological process exploration of top contributing pathways of discriminant factors)
- **GSEA** (Biological process exploration alongside EnrichR for discriminant factor contributors)

## Evaluation signals

- The MOFA model contains exactly ten factors after tuning, as configured in BiomiX
- Exactly five factors (1, 3, 4, 5 and 6) discriminate mutated from unmutated IGHV cohorts
- mofa_factor_annotation_cll.csv records correlations with the named clinical variables (11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13) for each discriminant factor
- Each discriminant factor's annotation row includes both EnrichR/GSEA biological process terms and PubMed literature hits for its top contributors
- Outliers were removed based on distances from PCA centroids and highly variable features were dropped before MOFA, per the Preview-QC interface

## Limitations

- BiomiX has limited types of omics analysers — proteomics and genomics are not yet supported and more integration methods remain to be implemented
- BiomiX is not a definitive solution; it is a first attempt to incorporate the gold standard single-omics pipeline, and users must carefully review datasets before use since there is no automated validation
- The Mac version has installation issues; Linux or Windows is recommended, and BiomiX does not support instrument normalization for metabolomics (QC samples must be labeled in the CONDITION column and removed manually or automatically after preview)

## Evidence
