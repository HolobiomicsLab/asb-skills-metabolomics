---
name: microbiome-differential-abundance-testing
description: Use when comparing compositional microbiome feature counts between AD
  model and control cohorts, e.g. testing KEGG pathway abundances in the 3xTg Sacrifice
  cohort against WT to reproduce Figure 5E.
when_to_use_negative:
- Input data is already a differential abundance result table rather than raw sequencing
  counts or a feature count table
- The comparison target is a metabolomics feature (e.g. phenylacetyl-carnitine) rather
  than a microbiome-derived feature; use the metabolomics differential pipeline instead
- Data comes from longitudinal fecal samples without animal IDs (collected randomly
  per cage), where single-animal developmental inference is not supported
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Qiita
  role: Public deposit used to retrieve the fecal shallow shotgun metagenomics sequencing
    data for 3xTg and WT Sacrifice cohorts prior to annotation and differential testing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:333d07dc8c9c004a12cf62166fd385112f5f10c8e9ad2b0b5a9f1abda1b10bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Qiita
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# microbiome-differential-abundance-testing

## Summary

Statistical framework for testing fecal metagenomics features (species, KEGG orthologs, pathways) for differential abundance between 3xTg AD mice and wild-type controls, using the same differential-testing approach applied at the species level via ANCOM-BC2 and extended to KEGG pathway profiles.

## When to use

Use when comparing compositional microbiome feature counts between AD model and control cohorts, e.g. testing KEGG pathway abundances in the 3xTg Sacrifice cohort against WT to reproduce Figure 5E.

## When NOT to use

- Input data is already a differential abundance result table rather than raw sequencing counts or a feature count table
- The comparison target is a metabolomics feature (e.g. phenylacetyl-carnitine) rather than a microbiome-derived feature; use the metabolomics differential pipeline instead
- Data comes from longitudinal fecal samples without animal IDs (collected randomly per cage), where single-animal developmental inference is not supported

## Inputs

- Fecal shallow shotgun metagenomics sequencing data (e.g. Qiita-deposited FASTQs, n=666 samples) from 3xTg and WT Sacrifice cohorts
- Sample metadata table linking each sequencing sample to genotype (3xTg vs WT), cohort (Sacrifice vs Longitudinal), and housing condition
- KEGG ortholog / pathway annotation files mapping reads or genes to KEGG pathways

## Outputs

- Differential abundance table with effect direction (enriched/depleted) and significance per pathway or species
- Combined pathway abundance and differential results table (e.g. kegg_pathway_enrichment_3xTg.tsv)

## How to apply

Retrieve raw sequencing data from the public Qiita deposit, then profile functional content by annotating reads to KEGG orthologs with a DIAMOND-based search, the same approach the paper used for gene searches. Aggregate ortholog assignments per sample into KEGG pathway abundance profiles. For each feature — whether a species or a pathway — apply a compositional differential-abundance test using ANCOM-BC2, the method the authors report for species-level analysis, and record both the direction of change and the significance call. For the 3xTg Sacrifice cohort (n=666 fecal shallow shotgun metagenomes), verify that nitrogen metabolism, glutathione metabolism, and toluene degradation are enriched while carotenoid biosynthesis is depleted in 3xTg animals relative to WT, as reported in Figure 5E. Output the combined pathway abundance and differential results as a table (e.g. kegg_pathway_enrichment_3xTg.tsv). Note the same statistical machinery applies to other microbiome feature types (species-level counts, functional orthologs), so reuse the differential-testing step across feature levels rather than switching methods per layer.

## Related tools

- **Qiita** (Public deposit used to retrieve the fecal shallow shotgun metagenomics sequencing data for 3xTg and WT Sacrifice cohorts prior to annotation and differential testing)

## Examples

```
Retrieve Qiita data with `qiita download-artifact --artifact-id <id>` and run ANCOM-BC2 on the pathway abundance table, e.g. `Rscript ancombc2_run.R --input kegg_pathways.tsv --metadata cohort_meta.tsv --group_col genotype`
```

## Evaluation signals

- Direction of change matches Figure 5E: nitrogen metabolism, glutathione metabolism, and toluene degradation are enriched in 3xTg animals; carotenoid biosynthesis is depleted
- Significance calls for the reported enriched/depleted pathways match the paper's reported significance where stated
- Output table contains both abundance values and differential statistics (direction and significance) for every tested pathway
- Number of samples analyzed matches the published cohort size (n=666 fecal shallow shotgun metagenomes)

## Limitations

- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound that could influence differential abundance calls
- Differences between Longitudinal and Sacrifice cohorts were observed for both microbiome and metabolomics data, attributed to differences in sample collection (fresh fecal vs. colon after sacrifice), so results may not transfer between cohorts
- Differential testing relies on the ANCOM-BC2 compositional model; other feature types (e.g. KEGG orthologs) may require the same compositional framework but additional normalization choices not fully specified in the article

## Evidence

- [results] KEGG pathway analysis of the 3xTg Sacrifice cohort metagenomes identified enrichment of nitrogen metabolism, glutathione metabolism, and toluene degradation, and depletion of carotenoid biosynthesis in 3xTg animals: [evidence span withheld — anchor status: not_found]
- [methods] Retrieve the SPF fecal shallow shotgun metagenomics sequencing data (n=666) for the 3xTg and WT Sacrifice cohorts from the public Qiita deposit.: [evidence span withheld — anchor status: not_found]
- [methods] Test each pathway for differential abundance between WT and 3xTg animals, following the same differential-testing approach used for species-level ANCOM-BC2 analysis.: [evidence span withheld — anchor status: not_found]
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
