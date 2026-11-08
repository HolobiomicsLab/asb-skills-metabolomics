---
name: serum-metabolomics-group-comparison
description: Use when you have a table of putatively identified metabolites with relative
  concentrations per sample (e.g. from eRah's idList/alignList output) split into
  two labeled groups and need per-metabolite fold-change and non-parametric significance
  statistics.
when_to_use_negative:
- The metabolite abundance table has not yet been produced by deconvolution/alignment/identification
  (run the eRah pipeline with deconvolveComp, alignComp, recMissComp, identifyComp
  first)
- More than two groups or paired/related samples are being compared — the Wilcoxon-Mann-Whitney
  rank-sum design here is strictly a two-group unpaired comparison
- Raw GC/MS chromatograms are the input rather than a post-identification quantification
  table; peak picking or deconvolution tools (XCMS, MetAlign, ChromaTOF, eRah) are
  the wrong layer
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: erah
  role: upstream tool that produced the putative metabolite identifications and relative
    concentrations fed into the two-group statistical comparison
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: environment used to compute per-metabolite Wilcoxon-Mann-Whitney rank-sum
    p-values and percent-of-variation and to assemble the output CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:d75db5401be39116f5444a0a5a0c23828d0bb319f523c5608383bcfc1f5b717b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# serum-metabolomics-group-comparison

## Summary

Compares relative concentrations of GC/MS-identified serum metabolites between two sample classes (HIAE patients vs healthy controls) by computing percent-of-variation and two-sided Wilcoxon-Mann-Whitney rank-sum p-values, as done for the eRah-identified dysregulated metabolites in Table 3 of the eRah paper.

## When to use

Use when you have a table of putatively identified metabolites with relative concentrations per sample (e.g. from eRah's idList/alignList output) split into two labeled groups and need per-metabolite fold-change and non-parametric significance statistics.

## When NOT to use

- The metabolite abundance table has not yet been produced by deconvolution/alignment/identification (run the eRah pipeline with deconvolveComp, alignComp, recMissComp, identifyComp first)
- More than two groups or paired/related samples are being compared — the Wilcoxon-Mann-Whitney rank-sum design here is strictly a two-group unpaired comparison
- Raw GC/MS chromatograms are the input rather than a post-identification quantification table; peak picking or deconvolution tools (XCMS, MetAlign, ChromaTOF, eRah) are the wrong layer

## Inputs

- eRah putative identifications with relative concentrations for the 25 serum samples (Supporting File 4), one row per metabolite (e.g. lactic acid, leucine 1TMS, leucine 2TMS, serine, 5-oxoproline, glutamic acid, ornithine, lysine)
- Sample class labels: 11 HIAE (hyperinsulinaemic androgen excess) patients vs 14 CTR (healthy control) serum samples

## Outputs

- table3_metabolite_stats.csv — one row per metabolite with name, percent of variation (100*(mean(HIAE)-mean(CTR))/mean(CTR)), and two-sided Wilcoxon-Mann-Whitney p-value

## How to apply

Load the eRah putative-identification output (Supporting File 4: relative concentrations for the 25 serum samples) and subset rows for the metabolites of interest, keeping all annotated forms such as leucine 1TMS and leucine 2TMS separately. Split relative concentrations by class label — here 11 HIAE patients vs 14 CTR controls. For each metabolite compute percent of variation as 100*(mean(HIAE) − mean(CTR))/mean(CTR), which captures direction and magnitude of dysregulation relative to controls. In R, compute a two-sided Wilcoxon-Mann-Whitney rank-sum test p-value per metabolite comparing the two groups; the non-parametric test is appropriate for small, possibly non-normal metabolite concentration distributions. Assemble metabolite name, percent variation, and p-value into a single table and export it (e.g. table3_metabolite_stats.csv). Validation: all eight recomputed p-values should fall below the reported significance threshold (p<0.01) and percent-variation values should match the direction and magnitude of the published Table 3.

## Related tools

- **erah** (upstream tool that produced the putative metabolite identifications and relative concentrations fed into the two-group statistical comparison) — http://CRAN.R-project.org/package=erah
- **R** (environment used to compute per-metabolite Wilcoxon-Mann-Whitney rank-sum p-values and percent-of-variation and to assemble the output CSV)

## Examples

```
Rscript -e 'stats <- t(sapply(mets, function(m) { x <- df[df$metabolite==m,]; c(100*(mean(x$conc[x$class=="HIAE"])-mean(x$conc[x$class=="CTR"]))/mean(x$conc[x$class=="CTR"]), wilcox.test(conc~class, data=x)$p.value) })); write.csv(stats, "table3_metabolite_stats.csv")'
```

## Evaluation signals

- All eight recomputed p-values (lactic acid, leucine, serine, 5-oxoproline, glutamic acid, ornithine, lysine) fall below the reported threshold p<0.01
- Percent-variation values match the direction (all elevated in HIAE vs CTR) and magnitude reported in Table 3 of the paper
- Group split is correct: 11 HIAE patients vs 14 CTR controls, totaling the 25 serum samples
- Output CSV has exactly three columns per spec: metabolite name, percent variation, p-value
- Leucine is handled as two separate annotated rows (1TMS and 2TMS) consistent with the eRah identification output

## Limitations

- Validation scope is limited to the 25 serum samples analyzed by GC-TOF MS in the HIAE comparative analysis
- Detailed HIAE comparison processing methods are deferred to supporting methods hosted externally at pubs.acs.org, not in the main text
- Significant dysregulation reported here is putative identification-level; the paper notes significantly dysregulated metabolites were further validated using pure standards and targeted analysis by GC-QqQ MS, LC-QqQ, and NMR

## Evidence

- [intro] We demonstrate the functionality of eRah through a comparative analysis of serum samples from adolescents with hyperinsulinaemic androgen excess: [evidence span withheld — anchor status: not_found]
- [abstract] eRah outputs a table with compound names, matching scores and integrated area of compounds for each sample: "eRah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [abstract] significantly dysregulated metabolites validated using pure standards and targeted analysis: "Significantly dysregulated metabolites are further validated using pure standards and targeted analysis by GC-triple quadrupole (QqQ) MS, LC-QqQ, and NMR."
- [methods] Serum aliquots (25 µL) were thawed at 4 ◦C ... supplemented with 20 µL of 1 µg/µL succinic-d4 acid: "Serum aliquots (25 µL) were thawed at 4 ◦C. Samples were briefly vortex-mixed and each aliquot was supplemented with 20 µL of 1 µg/µL succinic-d4 acid"
- [discussion] Validation scope limited to serum samples analyzed by GC-TOF MS: "The comparative analysis of serum samples by GC–TOF MS provided test data"
