---
name: pathway-enrichment-annotation
description: Use when a differential analysis or MOFA factor yields a significant
  feature list (e.g. |log2FC|>0.5, p.adj<0.1 peaks or top factor contributors) that
  must be interpreted via biological process enrichment and literature annotation.
when_to_use_negative:
- Input is a raw expression matrix or peak matrix with no prior differential analysis
  or factor decomposition — enrichment requires a pre-filtered significant feature
  list
- Proteomics or genomics data — the article notes BiomiX 'has limited types of omics
  analysers' and these modalities are not yet supported
- Features cannot be mapped to gene identifiers or annotated metabolites (no m/z annotation),
  so EnrichR/GSEA terms would be meaningless
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: BiomiX
  role: Graphical R-based interface that orchestrates single-omics pipelines and submits
    discriminant factor contributors to enrichment, clinical correlation, and PubMed
    annotation
  repo: https://github.com/IxI-97/BiomiX
- name: EnrichR
  role: Biological process enrichment of significant gene lists and MOFA factor contributors
  repo: https://github.com/IxI-97/BiomiX
- name: GSEA
  role: Biological process exploration to confirm enriched pathways among the significant
    feature set
  repo: https://github.com/IxI-97/BiomiX
- name: TidyMass
  role: Annotates metabolomics features using fragmentation spectra for enrichment
    input
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Provides the discriminant factors whose top contributors are annotated with
    pathway and literature terms
  repo: https://github.com/IxI-97/BiomiX
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:ed554883646715dad136eab07004983adf2b008e02f3ccd5ff9865e83e408fa7
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
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# pathway-enrichment-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate significant feature/gene lists from BiomiX single-omics or MOFA factor contributions with biological process enrichments using EnrichR and GSEA, and link top discriminant factor contributors to pathway terms, clinical correlations, and PubMed literature hits.

## When to use

Use when a differential analysis or MOFA factor yields a significant feature list (e.g. |log2FC|>0.5, p.adj<0.1 peaks or top factor contributors) that must be interpreted via biological process enrichment and literature annotation.

## When NOT to use

- Input is a raw expression matrix or peak matrix with no prior differential analysis or factor decomposition — enrichment requires a pre-filtered significant feature list
- Proteomics or genomics data — the article notes BiomiX 'has limited types of omics analysers' and these modalities are not yet supported
- Features cannot be mapped to gene identifiers or annotated metabolites (no m/z annotation), so EnrichR/GSEA terms would be meaningless

## Inputs

- Significant gene/feature lists from DESeq2/Limma or ChAMP differential analysis
- Significant metabolite peaks with |log2FC| > 0.5 and adjusted p < 0.1, annotated by m/z via CEU Mass Mediator
- Top contributing features of discriminant MOFA factors
- Clinical annotation table (e.g. IGHV status, trisomy 12, TP53/SF3B1/del17p13)

## Outputs

- EnrichR/GSEA biological process enrichment terms with FDR-adjusted p-values
- Per-factor annotation table combining clinical correlations, pathway terms, and PubMed literature hits (e.g. mofa_factor_annotation_cll.csv)
- Pathway enrichment table per subgroup (e.g. ifn_subgroup_enrichment.csv)

## How to apply

Run the single-omics pipelines first (DESeq2/Limma for transcriptomics, ChAMP for methylomics, TidyMass-based Wilcoxon testing with FDR correction for metabolomics) to obtain significant feature lists, or extract top contributors from the discriminant MOFA factors (e.g. the ten-factor CLL model with five discriminating factors for mutated vs unmutated IGHV). Submit the significant gene lists to EnrichR and GSEA for biological process exploration, as BiomiX integrates both; for metabolite lists, annotate peaks by m/z via the CEU Mass Mediator database and fragmentation spectra in TidyMass before enrichment (e.g. recovering sphingolipid signaling and retrograde endocannabinoid signaling in the PTB plasma study). For MOFA factors, additionally correlate discriminant factors with clinical annotations (e.g. 11q22.3 deletion, trisomy 12, TP53/SF3B1/del17p13) and use BiomiX's automated PubMed bibliography search, which returns the most related articles per top contributor. Compile clinical correlations, EnrichR/GSEA pathway terms, and literature hits per factor into an annotation table such as mofa_factor_annotation_cll.csv. Validate by confirming reported pathway terms are recovered among the enrichments of the significant feature set at FDR-adjusted p < 0.05.

## Related tools

- **BiomiX** (Graphical R-based interface that orchestrates single-omics pipelines and submits discriminant factor contributors to enrichment, clinical correlation, and PubMed annotation) — https://github.com/IxI-97/BiomiX
- **EnrichR** (Biological process enrichment of significant gene lists and MOFA factor contributors) — https://github.com/IxI-97/BiomiX
- **GSEA** (Biological process exploration to confirm enriched pathways among the significant feature set) — https://github.com/IxI-97/BiomiX
- **TidyMass** (Annotates metabolomics features using fragmentation spectra for enrichment input) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Provides the discriminant factors whose top contributors are annotated with pathway and literature terms) — https://github.com/IxI-97/BiomiX

## Evaluation signals

- Enrichment table contains significant hits (FDR-adjusted p-value below 0.05) for the expected terms, e.g. IFN signaling, nitric oxide biosynthetic process, and B cell receptor signaling
- Reported pathway terms (e.g. sphingolipid signaling pathway, retrograde endocannabinoid signaling) are recovered among the enrichments of the significant feature set
- Output table contains log2FC and FDR-adjusted p-value columns for every input peak, with adjusted p-values in [0,1]
- Annotation table records correlations with the named clinical variables plus EnrichR/GSEA pathway terms and PubMed literature hits for each discriminant factor

## Limitations

- Metabolite annotation depends on m/z matching in CEU Mass Mediator and fragmentation spectra in TidyMass; unannotated features cannot be enriched
- BiomiX is described as 'not a definitive solution; it is a first attempt to incorporate the gold standard single-omics pipeline', and users are advised to carefully review the dataset before use
- Proteomics and genomics modalities are not yet supported, limiting which feature lists can be generated for enrichment

## Evidence

- [readme] DIABLO is now fully embedded into the factor interpretation pipeline: [evidence span withheld — anchor status: not_found]
