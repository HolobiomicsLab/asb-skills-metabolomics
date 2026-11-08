---
name: metagenomic-functional-annotation
description: Use when you have shotgun metagenomics sequencing data (e.g. FASTQ from
  fecal samples) and need functional pathway enrichment comparisons between groups
  rather than only taxonomic composition.
when_to_use_negative:
- Input is already a pathway abundance or functional table — skip read-level annotation
  and go straight to differential testing.
- The goal is taxonomic (species-level) profiling only; use the taxonomic profiler
  instead of aggregating to KEGG pathways.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_3174
tools:
- name: Qiita
  role: Public repository from which the SPF fecal shallow shotgun metagenomics sequencing
    data for the 3xTg and WT Sacrifice cohorts are retrieved as the raw input for
    annotation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:bbe27dc51da7bca5faef7d23c9e928d2fde121ac117c55f9111c775af3d0f8d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Qiita
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metagenomic-functional-annotation

## Summary

Annotates metagenomic sequencing reads to functional gene families or orthologs (e.g. KEGG orthologs) and aggregates them into pathway abundance profiles so that functional, rather than only taxonomic, differences between host genotypes can be tested and visualized.

## When to use

Use when you have shotgun metagenomics sequencing data (e.g. FASTQ from fecal samples) and need functional pathway enrichment comparisons between groups rather than only taxonomic composition.

## When NOT to use

- Input is already a pathway abundance or functional table — skip read-level annotation and go straight to differential testing.
- The goal is taxonomic (species-level) profiling only; use the taxonomic profiler instead of aggregating to KEGG pathways.

## Inputs

- Shotgun metagenomics sequencing data (FASTQ) for SPF fecal samples, e.g. 3xTg and WT Sacrifice cohorts from the Qiita public deposit
- Sample metadata table with genotype/condition and cohort labels (WT vs 3xTg)

## Outputs

- Per-sample KEGG pathway abundance matrix (ortholog counts aggregated to pathway level)
- Differential enrichment/depletion summary table (kegg_pathway_enrichment_3xTg.tsv) with pathway, effect direction, and significance for 3xTg vs WT

## How to apply

Retrieve the raw shallow shotgun sequencing data (here, SPF fecal samples, n=666, 3xTg and WT Sacrifice cohorts, from the public Qiita deposit) and annotate individual sequencing reads to KEGG orthologs using a DIAMOND-based gene search, the same profiler the study used for species-level searches. Aggregate ortholog assignments into KEGG pathway abundance profiles per sample. Test each pathway for differential abundance between WT and 3xTg animals using the same differential-testing approach applied for species-level ANCOM-BC2 analysis, then summarize enriched vs. depleted pathways with effect direction and significance. This moves the comparison from taxonomic composition (Akkermansia depleted, Helicobacter/Desulfovibrio enriched in 3xTg) to functional potential, enabling direct cross-cohort pathway interpretation.

## Related tools

- **Qiita** (Public repository from which the SPF fecal shallow shotgun metagenomics sequencing data for the 3xTg and WT Sacrifice cohorts are retrieved as the raw input for annotation.)

## Evaluation signals

- Output table reproduces the reported direction of change for key pathways — nitrogen metabolism, glutathione metabolism, and toluene degradation enriched, and carotenoid biosynthesis depleted in 3xTg animals (Figure 5E).
- Per-sample pathway abundance matrix has non-zero coverage for all annotated samples; samples with zero ortholog hits indicate a failed annotation run.
- Significance calls match those reported in the article where pathways were reported as significant.
- Aggregation from KEGG orthologs to pathways preserves the number of contributing reads (sum of ortholog counts per pathway equals the pathway abundance).
- Comparing 3xTg vs WT pathway directionality is consistent with the species-level findings (e.g. enrichment of Desulfovibrio and Helicobacter aligns with sulfur/glutathione-related pathway shifts).

## Limitations

- Shallow shotgun depth limits detection of low-abundance genes and pathways, so absence of an annotation does not mean absence of function.
- Functional profiles are inferred from sequencing reads via ortholog matching; strain-level variation and incomplete KEGG reference coverage may bias pathway aggregation.
- Differences between Longitudinal and Sacrifice cohorts (fresh fecal vs. colon after sacrifice) confound direct cross-cohort pathway comparisons.

## Evidence

- [intro] SPF fecal shallow shotgun metagenomics sequencing data (n=666) for the 3xTg and WT Sacrifice cohorts: [evidence span withheld — anchor status: not_found]
- [results] KEGG pathway analysis of the 3xTg Sacrifice cohort metagenomes identified enrichment of nitrogen metabolism, glutathione metabolism, and toluene degradation, and depletion of carotenoid biosynthesis in 3xTg animals.: "Systems-level analyses of 3xTg and 5xFAD mice reveal clusters of dysregulated molecular classes across tissues including carnitines, bile acids, B vitamins, and neurotransmitters"
- [discussion] Differences between Longitudinal and Sacrifice cohorts ... attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
