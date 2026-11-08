---
name: gwas-summary-statistics-processing
description: Use when published GWAS summary statistics for multiple traits must be
  harmonized into LDSC-format annotations (e.g., SNPs within 100kb of sex-consistent
  8-week DE genes per tissue) to estimate heritability enrichment per trait x tissue.
when_to_use_negative:
- The question is the proportion of hSNP2 mediated by gene expression rather than
  annotation-level heritability enrichment — use MESC with GWAS plus eQTL summary
  statistics instead
- Only individual-level genotype/phenotype data are available rather than GWAS summary
  statistics
- Trait and tissue gene-set sizes are too small for stable LDSC regression (few tags
  in the annotation)
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: LDSC
  role: Estimates SNP-heritability proportions and enrichment from GWAS summary statistics
    given tissue DE gene-set annotations; version 1.0.1, run per the 'Cell type specific
    analyses' tutorial
  repo: https://github.com/bulik/ldsc
- name: MESC
  role: Complementary tool incorporating GWAS and eQTL summary statistics to estimate
    the proportion of hSNP2 mediated by gene expression within and across tissues
  repo: https://github.com/douglasyao/mesc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:1c9f75b8e1a635f8277066c19b889899227e07f8d020cc8bb6543428f54b57a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bulik/ldsc
  tools:
  - LDSC
  - MESC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gwas-summary-statistics-processing

## Summary

Process publicly available GWAS summary statistics (114 traits/diseases) with LDSC 1.0.1 to estimate SNP-heritability (hSNP2) proportions and enrichment captured by tissue-specific exercise-adaptation gene-set annotations, benchmarked against a random gene-set negative control.

## When to use

Use when published GWAS summary statistics for multiple traits must be harmonized into LDSC-format annotations (e.g., SNPs within 100kb of sex-consistent 8-week DE genes per tissue) to estimate heritability enrichment per trait x tissue.

## When NOT to use

- The question is the proportion of hSNP2 mediated by gene expression rather than annotation-level heritability enrichment — use MESC with GWAS plus eQTL summary statistics instead
- Only individual-level genotype/phenotype data are available rather than GWAS summary statistics
- Trait and tissue gene-set sizes are too small for stable LDSC regression (few tags in the annotation)

## Inputs

- Published GWAS summary statistics for 114 traits and diseases
- Sex-consistent 8-week DE gene sets per tissue (5% IHW alpha, F1_M1 / F-1_M-1 states)
- LDSC baseline annotation (53 functional categories)
- Per-tissue all-expressed-genes annotation (100kb windows)
- LD Scores compatible with the GWAS ancestry panel

## Outputs

- Per-trait, per-tissue LDSC heritability enrichment proportions, enrichment factors, and p-values (CSV)
- Negative-control null p-value distributions from randomly sampled gene sets of equivalent size
- Conditional enrichment estimates adjusting for other tissues' gene sets

## How to apply

Retrieve the published GWAS summary statistics for the 114 traits and build annotations of loci within 100kb windows of sex-consistent 8-week DE genes per tissue, alongside the LDSC baseline annotation (53 functional categories) and an annotation of loci within 100kb of all expressed genes per tissue. Munge the summary statistics and run LDSC 1.0.1 following the 'Cell type specific analyses' tutorial to estimate the proportion of and enrichment in SNP-heritability (hSNP2) for each trait x tissue DE gene set. Re-run LDSC with all other tissues' annotations included to obtain enrichment conditional on tissue-gene-set overlaps. As a negative control, sample random gene sets of equivalent size per tissue and re-run LDSC to derive null p-value distributions. Compile per-trait, per-tissue enrichment proportions, enrichment factors, and p-values into CSV output tables.

## Related tools

- **LDSC** (Estimates SNP-heritability proportions and enrichment from GWAS summary statistics given tissue DE gene-set annotations; version 1.0.1, run per the 'Cell type specific analyses' tutorial) — https://github.com/bulik/ldsc
- **MESC** (Complementary tool incorporating GWAS and eQTL summary statistics to estimate the proportion of hSNP2 mediated by gene expression within and across tissues) — https://github.com/douglasyao/mesc

## Examples

```
git clone https://github.com/bulik/ldsc.git && python ldsc.py --h2 trait_sumstats.txt.gz --ref-ld-chr baseline,exercise_DE_100kb --w-ld-chr weights.hm3_noMHA --out trait_x_tissue_enrichment
```

## Evaluation signals

- Significant LDSC heritability enrichments should appear in 43 traits at Bonferroni-adjusted alpha = 0.05 across tissue-specific 8-week sex-consistent DE gene sets
- The largest enrichment factor should most often correspond to the spleen (22/43 ≈ 51% of traits) with an average enrichment factor of ≈2.85
- Roughly 10% of trait heritability should be captured by the top-enriched gene sets (e.g., spleen DE genes for asthma at nearly four times baseline expectation)
- Randomly sampled gene sets of equivalent size should yield highly similar p-value distributions, confirming the null benchmark
- CSV output tables must contain enrichment proportion, enrichment factor, and p-value columns for every trait x tissue pair plus the simulation nulls

## Limitations

- Simulation with randomly sampled gene sets of equivalent size produced highly similar p-value distributions, so enrichment results are best interpreted descriptively rather than as strong biological signals
- GTEx sample comprises mostly European-descendant individuals, so results have limited portability to non-European populations
- Rat-human orthology is imperfect (nearly 140 million years of evolution), so important biology in disease/exercise-responsive genes without easy cross-species correspondence is likely missed

## Evidence

- [intro] We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes: "We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes"
- [intro] GWAS 114 traits: "previously connected to 114 traits and diseases from publicly available Genome Wide Association Studies (GWAS) distributed across several phenotypic"
- [methods] LDSC version and repo: "[LDSC](https://github.com/bulik/ldsc) (1.0.1)"
- [discussion] Spleen enrichment finding: [evidence span withheld — anchor status: not_found]
- [readme] LDSC tool description: [evidence span withheld — anchor status: not_found]
