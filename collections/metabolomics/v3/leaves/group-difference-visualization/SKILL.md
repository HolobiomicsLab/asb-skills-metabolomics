---
name: group-difference-visualization
description: Use when KEGG pathway abundance profiles must be compared between genotype
  groups (e.g. 3xTg vs WT fecal metagenomes) to identify and visualize enriched vs
  depleted pathways with effect direction and significance.
when_to_use_negative:
- The question concerns species-level taxonomic differences (e.g. Akkermansia muciniphila
  depletion, Mucispirillum schaedleri enrichment) rather than functional pathway contrasts
  — use species-level ANCOM-BC2 directly instead
- Input data are untargeted metabolomics features (carnitines, bile acids, B vitamins)
  rather than metagenomic functional profiles
- Samples come from the Longitudinal fecal cohort, where samples were collected randomly
  per cage without animal IDs, preventing single-animal genotype-paired testing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_3174
tools:
- name: Qiita
  role: Source of the public SPF fecal shallow shotgun metagenomics deposit (n=666)
    for the 3xTg and WT Sacrifice cohorts
- name: DIAMOND-based functional profiler
  role: Annotates sequencing reads to KEGG orthologs before aggregation into pathway
    abundance profiles
- name: ANCOM-BC2
  role: Differential-testing approach applied to pathway abundances, mirroring the
    species-level analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:4e86ec541c20fdaf6923ef360a3868dbc57b6fcb7205c9e49aa49faab3477dba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC2
  - DIAMOND-based functional profiler
  - Qiita
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# group-difference-visualization

## Summary

Summarizes and visualizes differential KEGG pathway abundance between experimental groups — here 3xTg vs WT Sacrifice-cohort fecal metagenomes — producing an enriched/depleted summary table (Figure 5E-style) with effect direction and significance per pathway.

## When to use

Use when KEGG pathway abundance profiles must be compared between genotype groups (e.g. 3xTg vs WT fecal metagenomes) to identify and visualize enriched vs depleted pathways with effect direction and significance.

## When NOT to use

- The question concerns species-level taxonomic differences (e.g. Akkermansia muciniphila depletion, Mucispirillum schaedleri enrichment) rather than functional pathway contrasts — use species-level ANCOM-BC2 directly instead
- Input data are untargeted metabolomics features (carnitines, bile acids, B vitamins) rather than metagenomic functional profiles
- Samples come from the Longitudinal fecal cohort, where samples were collected randomly per cage without animal IDs, preventing single-animal genotype-paired testing

## Inputs

- SPF fecal shallow shotgun metagenomics sequencing data (n=666) for 3xTg and WT Sacrifice cohorts, retrieved from the public Qiita deposit
- Sample metadata mapping samples to genotype (WT vs 3xTg) and cohort (Sacrifice)
- KEGG ortholog and pathway reference mappings for aggregating KO annotations into pathway profiles

## Outputs

- Summary table kegg_pathway_enrichment_3xTg.tsv listing each KEGG pathway with effect direction (enriched or depleted in 3xTg) and significance
- Figure 5E-style visualization of enriched vs depleted KEGG pathways in 3xTg animals

## How to apply

Retrieve the SPF fecal shallow shotgun metagenomics sequencing data (n=666) for the 3xTg and WT Sacrifice cohorts from the public Qiita deposit. Annotate sequencing reads to KEGG orthologs using a standard metagenomic functional profiler (a DIAMOND-based search, as used in the study for gene searches), then aggregate ortholog assignments into per-sample KEGG pathway abundance profiles. Test each pathway for differential abundance between WT and 3xTg animals using the same differential-testing approach applied to species-level ANCOM-BC2 analysis. Generate a summary table of enriched and depleted KEGG pathways with effect direction and significance (kegg_pathway_enrichment_3xTg.tsv), and render the enriched-vs-depleted contrast as a Figure 5E-style group-difference visualization.

## Related tools

- **Qiita** (Source of the public SPF fecal shallow shotgun metagenomics deposit (n=666) for the 3xTg and WT Sacrifice cohorts)
- **DIAMOND-based functional profiler** (Annotates sequencing reads to KEGG orthologs before aggregation into pathway abundance profiles)
- **ANCOM-BC2** (Differential-testing approach applied to pathway abundances, mirroring the species-level analysis)

## Evaluation signals

- The output table reproduces the reported direction of change: nitrogen metabolism, glutathione metabolism, and toluene degradation enriched, and carotenoid biosynthesis depleted, in 3xTg animals (Figure 5E)
- Significance calls for the reported pathways match those in Figure 5E
- Every tested KEGG pathway row carries both an effect-direction label and a significance value
- Pathway profiles derive from DIAMOND-based KO annotation of the 666-sample Qiita deposit, not from a precomputed functional table
- Differential testing uses the same statistical framework as the species-level ANCOM-BC2 analysis of the Sacrifice cohort

## Limitations

- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound for genotype-contrast interpretation
- Differences between Longitudinal and Sacrifice cohorts in microbiome data are attributed to sample collection differences (fresh fecal vs colon after sacrifice), so pathway results should not be extrapolated across cohorts
- Shallow shotgun sequencing limits functional resolution of KO-level annotation feeding the pathway profiles

## Evidence

- [abstract] complemented by shotgun metagenomics sequencing data (666 samples): "complemented by shotgun metagenomics sequencing data (666 samples)"
- [intro] 3xTg animals were characterized by depletion of Akkermansia muciniphila and an enrichment of Mucispirillum schaedleri, Helicobacter, Desulfovibrio, Escherichia, and Shigella species: "3xTg animals were characterized by depletion of Akkermansia muciniphila and an enrichment of Mucispirillum schaedleri, Helicobacter, Desulfovibrio, Escherichia, and Shigella species"
- [abstract] This signature, coupled with microbiome profiles, suggests increased oxidative stress via mitochondrial dysfunction: "This signature, coupled with microbiome profiles, suggests increased oxidative stress via mitochondrial dysfunction"
