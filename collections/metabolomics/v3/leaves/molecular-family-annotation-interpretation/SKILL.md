---
name: molecular-family-annotation-interpretation
description: Use when a feature quantification table plus MGF MS/MS spectra must be
  grouped into molecular families (e.g. GNPS FBMN) and scored per phenotype to identify
  differentially enriched, annotated families.
when_to_use_negative:
- The input is gene cluster abundance/expression data from metagenomes or metatranscriptomes
  rather than metabolomics feature tables — use BiG-MAP's abundance/expression profiling
  modules instead.
- Only a feature table is available without MS/MS fragmentation spectra (no MGF),
  so molecular families cannot be built by fragmentation similarity.
- Molecular families have already been clustered and activity-scored and only a phenotype-annotation
  narrative is required.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: BiG-MAP
  role: Provides the `bigmap-download` entry point used to fetch the processed spectra,
    feature quantification table, and metadata for the MSV000081832 reanalysis; its
    published reuterin-MGC/(poly)acrolein finding is the meta'omics context for interpreting
    the enriched molecular family.
  repo: https://github.com/medema-group/BiG-MAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:078beff299d90cab9f292c3210591fe7caf10632c7debbc397dd5750266b6129
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# molecular-family-annotation-interpretation

## Summary

Clusters metabolomics features into molecular families from MS/MS fragmentation similarity and interprets which families are enriched in a phenotype, connecting metabolite-level signals (e.g. (poly)acrolein-like fragmentation) to gene-cluster-level findings such as reuterin MGC enrichment.

## When to use

Use when a feature quantification table plus MGF MS/MS spectra must be grouped into molecular families (e.g. GNPS FBMN) and scored per phenotype to identify differentially enriched, annotated families.

## When NOT to use

- The input is gene cluster abundance/expression data from metagenomes or metatranscriptomes rather than metabolomics feature tables — use BiG-MAP's abundance/expression profiling modules instead.
- Only a feature table is available without MS/MS fragmentation spectra (no MGF), so molecular families cannot be built by fragmentation similarity.
- Molecular families have already been clustered and activity-scored and only a phenotype-annotation narrative is required.

## Inputs

- Processed MGF spectra file from the metabolomics study (GNPS-MassIVE MSV000081832)
- Feature quantification table (features x samples)
- Sample metadata / phenotype file (25 healthy vs 24 caries oral samples)
- GNPS-MassIVE accession list for data download

## Outputs

- Molecular families (FBMN clusters of features grouped by spectral fragmentation similarity)
- Per-family PALS activity score table (pals_activity_scores.csv)
- Acrolein-family enrichment summary (acrolein_family_summary.csv)

## How to apply

Fetch the processed MGF spectra, feature quantification table, and sample metadata for the study (e.g. GNPS-MassIVE accession MSV000081832), using a download entry point such as BiG-MAP's `bigmap-download` where applicable. Run Feature-Based Molecular Networking (FBMN, GNPS workflow) on the MGF spectra and feature table so that features are clustered into molecular families by spectral fragmentation similarity. Then run PALS (Pathway Activity Level Scoring) on the feature table with the phenotype metadata — here 25 healthy vs 24 caries oral samples — to compute set-based activity scores per molecular family. Rank families by PALS activity score for the phenotype of interest (healthy) and extract families whose member features carry polymer-like (poly)acrolein annotations. Interpret any enriched family jointly with the meta'omics result: in this study, metabolomic masses with fragmentation patterns consistent with (poly)acrolein came from the same samples in which a putative reuterin biosynthetic gene cluster from a Streptococcus strain was enriched, since acrolein is known to form from reuterin pathway products and inhibit pathogenic Streptococcus mutans.

## Related tools

- **BiG-MAP** (Provides the `bigmap-download` entry point used to fetch the processed spectra, feature quantification table, and metadata for the MSV000081832 reanalysis; its published reuterin-MGC/(poly)acrolein finding is the meta'omics context for interpreting the enriched molecular family.) — https://github.com/medema-group/BiG-MAP

## Examples

```
bigmap-download -A msv000081832_accessions.txt -O ./msv000081832/
```

## Evaluation signals

- The PALS output table (pals_activity_scores.csv) contains per-family activity scores and the (poly)acrolein-annotated molecular family ranks as enriched in healthy samples.
- acrolein_family_summary.csv reports one differentially expressed molecular family with three candidate metabolites annotated as polyacrolein-like, enriched in healthy samples.
- The analysis reproduces the paper's co-occurrence pattern: acrolein-consistent metabolomic masses come from the same samples in which the reuterin biosynthetic gene cluster was enriched.
- Phenotype grouping matches the source cohort: 25 healthy vs 24 caries samples in the PALS phenotype metadata.
- FBMN output groups features into families using spectral fragmentation similarity rather than arbitrary feature grouping.

## Limitations

- The reuterin gene cluster was 'enriched but not exclusively found in healthy samples', so the metabolite-family enrichment does not perfectly partition phenotypes.
- (Poly)acrolein annotation is based on fragmentation-pattern consistency, not confirmed structure, so candidates remain putative.
- BiG-MAP itself 'has been only tested on Ubuntu', so downstream download tooling may need that environment.

## Evidence

- [abstract] a putative reuterin biosynthetic gene cluster from a Streptococcus strain to be enriched but not exclusively found in healthy samples; metabolomic data from the same samples showed masses: "we found a putative reuterin biosynthetic gene cluster from a Streptococcus strain to be enriched but not exclusively found in healthy samples; metabolomic data from the same samples showed masses"
- [abstract] BiG-MAP profiles gene cluster abundance/expression and differential analysis across conditions, anchoring the meta'omics side of the metabolite-family interpretation.: "BiG-MAP, a bioinformatic tool to profile abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluate their differential abundance and expression under"
- [readme] bigmap-download fetches study data by accession list, the entry point used to retrieve the MSV000081832 metabolomics files for reanalysis.: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-MAP environment constraint caveat from the README.: [evidence span withheld — anchor status: external_doc]
