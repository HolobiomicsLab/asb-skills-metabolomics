---
name: conditional-heritability-enrichment-across-tissues
description: Use when a tissue-specific gene-set annotation shows significant LDSC
  heritability enrichment and you must test whether that signal is independent of
  overlapping gene sets from other tissues.
when_to_use_negative:
- When the question is expression-mediated heritability (proportion of hSNP2 mediated
  by gene expression) rather than partitioned heritability enrichment — MESC, not
  conditional LDSC, is the right tool.
- When gene sets are only marginally significant and no cross-tissue overlap exists
  to justify conditioning; marginal (unconditional) LDSC enrichment suffices.
- When GWAS summary statistics lack appropriate sample-size/allele-frequency metadata
  or a matching LD reference, since LDSC cannot be validly applied.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3517
tools:
- name: LDSC 1.0.1
  role: Estimates partitioned SNP-heritability (hSNP2) and enrichment for each trait
    x tissue DE gene-set annotation, run conditionally with all other tissue annotations
    alongside the 53-category baseline
  repo: https://github.com/bulik/ldsc
- name: MESC
  role: Complementary tool estimating the proportion of hSNP2 mediated by assayed
    gene expression using GWAS and eQTL summary statistics
  repo: https://github.com/douglasyao/mesc
- name: GCTA 1.93.2 (GCTA-GRM)
  role: Estimates hSNP2 of each gene's expression as input for downstream heritability
    analyses
  repo: https://yanglab.westlake.edu.cn/software/gcta/#Overview
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:802ad18c37441eaaab1f4f61cd56454fd4209d5b8a016e3fd95ce2af52d0c22d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bulik/ldsc
  tools:
  - GCTA 1.93.2 (GCTA-GRM)
  - LDSC 1.0.1
  - MESC
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# conditional-heritability-enrichment-across-tissues

## Summary

Re-runs LDSC with the focal tissue's exercise-DE gene-set annotation entered alongside annotations for all other tissues (plus the 53-category baseline) so that per-tissue SNP-heritability enrichment is estimated conditional on tissue-gene-set overlaps, disambiguating which tissue's DE genes truly drive hSNP2 enrichment.

## When to use

Use when a tissue-specific gene-set annotation shows significant LDSC heritability enrichment and you must test whether that signal is independent of overlapping gene sets from other tissues.

## When NOT to use

- When the question is expression-mediated heritability (proportion of hSNP2 mediated by gene expression) rather than partitioned heritability enrichment — MESC, not conditional LDSC, is the right tool.
- When gene sets are only marginally significant and no cross-tissue overlap exists to justify conditioning; marginal (unconditional) LDSC enrichment suffices.
- When GWAS summary statistics lack appropriate sample-size/allele-frequency metadata or a matching LD reference, since LDSC cannot be validly applied.

## Inputs

- Published GWAS summary statistics for 114 traits and diseases
- Per-tissue binary annotations of SNPs within 100kb of sex-consistent 8-week DE genes (repfdr F1_M1 / F-1_M-1 states, 5% IHW alpha)
- LDSC baseline annotation (53 functional categories) with matching LD scores
- Annotation of loci within 100kb of all expressed genes per tissue
- LD score and weight files compatible with the GWAS panel (e.g. 1000G EUR)

## Outputs

- Per-trait, per-tissue conditional enrichment factors, proportions of hSNP2 (~10% for spleen), and p-values
- Null p-value distributions from randomly sampled gene-set simulations of equivalent size
- CSV tables compiling enrichment proportions, enrichment factors, and p-values per trait x tissue

## How to apply

First build binary annotations of SNPs within 100kb windows of the sex-consistent 8-week DE genes for each tissue (selected via repfdr F1_M1 / F-1_M-1 states at 5% IHW alpha), alongside the LDSC baseline model (53 functional categories) and an annotation of loci within 100kb of all expressed genes per tissue. Run LDSC 1.0.1 following the 'Cell type specific analyses' tutorial to obtain marginal proportion of hSNP2 and enrichment per trait x tissue DE gene set. Then re-run LDSC including annotations for all other tissues' DE gene sets together with the baseline, yielding heritability enrichment for each tissue conditional on the others — this controls for the substantial locus overlap between tissue gene sets. Interpret enrichment factors (~2.85 average, spleen largest in 22/43 traits) against a negative-control simulation of randomly sampled gene sets of equivalent size, since random sets produced highly similar p-value distributions, so report results descriptively rather than as definitive causal evidence.

## Related tools

- **LDSC 1.0.1** (Estimates partitioned SNP-heritability (hSNP2) and enrichment for each trait x tissue DE gene-set annotation, run conditionally with all other tissue annotations alongside the 53-category baseline) — https://github.com/bulik/ldsc
- **MESC** (Complementary tool estimating the proportion of hSNP2 mediated by assayed gene expression using GWAS and eQTL summary statistics) — https://github.com/douglasyao/mesc
- **GCTA 1.93.2 (GCTA-GRM)** (Estimates hSNP2 of each gene's expression as input for downstream heritability analyses) — https://yanglab.westlake.edu.cn/software/gcta/#Overview

## Examples

```
python ldsc.py --h2 trait.sumstats.gz --ref-ld-chr baseline,spleen_de,other_tissues --w-ld-chr weights.hm3_noMHC --frqfile 1000G.EUR.QC --out trait_spleen_conditional
```

## Evaluation signals

- Reproduced results show significant enrichments in 43 traits at Bonferroni-adjusted alpha = 0.05 across tissue-specific 8-week sex-consistent DE gene sets.
- The largest enrichment factor corresponds to the spleen for 22/43 (~51%) of traits, with an average enrichment factor of ~2.85.
- Spleen DE genes account for ~10% of trait heritability, nearly four times baseline expectation (validated for asthma).
- Randomly sampled gene sets of equivalent size yield p-value distributions highly similar to real sets — confirming results should be read descriptively.
- Output CSVs contain per-trait, per-tissue enrichment proportions, enrichment factors, p-values, and the simulation p-value distributions.

## Limitations

- Simulation experiments with randomly sampled gene sets of equivalent size produced highly similar p-value distributions, so enrichment results are best interpreted descriptively rather than as strong causal evidence.
- Rat (MoTrPAC) DE genes must be mapped to human orthologs; nearly 140 million years of evolution separate rats and humans, complicating interpretation.
- GTEx-based results have limited portability to non-European populations, as the GTEx sample comprises mostly European-descendant individuals.

## Evidence

- [methods] Re-run LDSC with annotations for all other tissues included alongside the baseline annotation to obtain heritability and heritability enrichment conditional on tissue-gene-set overlaps.: [evidence span withheld — anchor status: not_found]
- [intro] LDSC accommodates LD to estimate SNP-heritability captured by exercise adaptation gene sets.: "We applied LDSC9, which can accommodate linkage disequilibrium to estimate SNP-heritability captured by sets of exercise adaption genes"
- [discussion] Spleen DE genes show greatest hSNP2 enrichment, nearly four times baseline, ~10% of trait heritability.: [evidence span withheld — anchor status: not_found]
- [readme] LDSC v1.0.1 is a command line tool for estimating heritability and genetic correlation from GWAS summary statistics.: [evidence span withheld — anchor status: not_found]
- [methods] Training-differential genes were selected at 5% IHW alpha.: "Training-differential genes were selected at 5% IHW α."
