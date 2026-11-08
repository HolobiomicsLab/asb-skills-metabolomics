---
name: kegg-pathway-abundance-profiling
description: Use when you have raw fecal shallow shotgun metagenomics sequencing data
  (e.g. the n=666 3xTg/WT Sacrifice cohort deposit on Qiita) and need per-sample KEGG
  pathway abundances plus genotype-level enrichment/depletion calls.
when_to_use_negative:
- The input is already a KO- or pathway-level abundance feature table — re-annotating
  raw reads with DIAMOND would be redundant; proceed directly to aggregation and differential
  testing
- The goal is taxonomic profiling of species (e.g. the species-level ANCOM-BC2 analysis
  of Akkermansia muciniphila depletion and Mucispirillum/Helicobacter/Desulfovibrio/Escherichia/Shigella
  enrichment) rather than functional pathway profiling
- The samples come from the Longitudinal cohort, where fecal samples lack animal IDs
  and collection differences confound comparison with the Sacrifice cohort
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: Qiita
  role: Public repository hosting the SPF fecal shallow shotgun metagenomics sequencing
    data (n=666) for the 3xTg and WT Sacrifice cohorts used as the input
- name: DIAMOND
  role: DIAMOND-based search, as used in the study for gene searches, assigns sequencing
    reads to KEGG orthologs prior to pathway aggregation
- name: ANCOM-BC2
  role: Differential-testing framework whose species-level approach in the study is
    reused to test each KEGG pathway for differential abundance between WT and 3xTg
    animals
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:8fad904bbdd1e01bb1fb0b5c096ffdfd9225efee2aec17c52f63dcf40ab936fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC2
  - DIAMOND
  - Qiita
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kegg-pathway-abundance-profiling

## Summary

Annotates raw shallow shotgun metagenomics reads to KEGG orthologs, aggregates ortholog counts into per-sample KEGG pathway abundance profiles, and tests each pathway for differential abundance between genotype groups (here, 3xTg vs WT fecal metagenomes from the Sacrifice cohort), yielding an enriched/depleted pathway table.

## When to use

Use when you have raw fecal shallow shotgun metagenomics sequencing data (e.g. the n=666 3xTg/WT Sacrifice cohort deposit on Qiita) and need per-sample KEGG pathway abundances plus genotype-level enrichment/depletion calls.

## When NOT to use

- The input is already a KO- or pathway-level abundance feature table — re-annotating raw reads with DIAMOND would be redundant; proceed directly to aggregation and differential testing
- The goal is taxonomic profiling of species (e.g. the species-level ANCOM-BC2 analysis of Akkermansia muciniphila depletion and Mucispirillum/Helicobacter/Desulfovibrio/Escherichia/Shigella enrichment) rather than functional pathway profiling
- The samples come from the Longitudinal cohort, where fecal samples lack animal IDs and collection differences confound comparison with the Sacrifice cohort

## Inputs

- Raw shallow shotgun metagenomics sequencing reads (FASTQ) from SPF fecal samples of the 3xTg and WT Sacrifice cohorts, retrieved from the public Qiita deposit
- Sample metadata (genotype 3xTg vs WT, cohort = Sacrifice) for grouping and differential testing
- KEGG ortholog-to-pathway mapping used to aggregate KO annotations into pathway abundance profiles

## Outputs

- Per-sample KEGG pathway abundance profile matrix
- Differential abundance results table of KEGG pathways with effect direction (enriched/depleted in 3xTg) and significance calls
- Combined pathway abundance and differential results table written to kegg_pathway_enrichment_3xTg.tsv

## How to apply

Retrieve the SPF fecal shallow shotgun metagenomics sequencing data (n=666) for the 3xTg and WT Sacrifice cohorts from the public Qiita deposit. Annotate sequencing reads to KEGG orthologs using a standard metagenomic functional profiler with a DIAMOND-based search, as used in the study for gene searches. Aggregate ortholog assignments into per-sample KEGG pathway abundance profiles. Test each pathway for differential abundance between WT and 3xTg animals with the same differential-testing approach used for the species-level ANCOM-BC2 analysis, so that compositional effects are handled consistently. Summarize enriched and depleted pathways with effect direction and significance, and write the combined pathway abundance plus differential results table to kegg_pathway_enrichment_3xTg.tsv.

## Related tools

- **Qiita** (Public repository hosting the SPF fecal shallow shotgun metagenomics sequencing data (n=666) for the 3xTg and WT Sacrifice cohorts used as the input)
- **DIAMOND** (DIAMOND-based search, as used in the study for gene searches, assigns sequencing reads to KEGG orthologs prior to pathway aggregation)
- **ANCOM-BC2** (Differential-testing framework whose species-level approach in the study is reused to test each KEGG pathway for differential abundance between WT and 3xTg animals)

## Evaluation signals

- The output table kegg_pathway_enrichment_3xTg.tsv reproduces the reported direction of change (Figure 5E): nitrogen metabolism, glutathione metabolism, and toluene degradation enriched, and carotenoid biosynthesis depleted in 3xTg animals
- Significance calls for the reported pathways match those in Figure 5E where the study reports them
- Every sample row in the pathway abundance table aggregates from KO-level assignments with no missing per-sample profiles across the n=666 metagenomes
- Cohort metadata confirms only Sacrifice-cohort SPF samples were included, excluding Longitudinal-cohort fecal samples
- Differential testing is composition-aware (same approach as the species-level ANCOM-BC2 analysis), not naive per-pathway t-testing

## Limitations

- Differences between the Longitudinal and Sacrifice cohorts in microbiome and metabolomics data were observed and attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so results are specific to Sacrifice-cohort samples
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound for genotype-attributed pathway differences
- Hundreds of non-annotated molecules/features can remain uncharacterized, and annotation completeness limits which functional signals the pathway profile captures

## Evidence

- [abstract] shallow shotgun metagenomics sequencing data (666 samples): "complemented by shotgun metagenomics sequencing data (666 samples)"
- [intro] 3xTg microbiome depletion/enrichment pattern: "3xTg animals were characterized by depletion of Akkermansia muciniphila and an enrichment of Mucispirillum schaedleri, Helicobacter, Desulfovibrio, Escherichia, and Shigella species"
- [discussion] cohort effect confound: "3xTg and WT mice were bred and housed separately before the start of the experiment"
- [intro] centralized dataset including paired fecal metagenomics from SPF cohorts: "we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples – including brain, serum, liver, cecal content, and feces – from 307 animals"
