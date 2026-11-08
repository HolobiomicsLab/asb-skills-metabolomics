---
name: metabolite-percent-variation-calculation
description: Use when an eRah deconvolution/alignment run yields relative concentrations
  of putatively identified compounds for two sample groups (e.g. HIAE vs CTR) and
  you need per-metabolite percent variation and rank-sum significance to report dysregulated
  metabolites.
when_to_use_negative:
- Input is already a processed feature table of fold changes or log2 ratios — percent-variation
  recomputation would be redundant
- Data are raw GC/MS chromatograms that have not yet been deconvolved and aligned
  by eRah (deconvolveComp, alignComp, identifyComp must run first)
- More than two sample classes or paired/replicate designs that violate the two-sample
  Wilcoxon-Mann-Whitney assumption
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: erah
  role: Produces the deconvolved, aligned and identified compound table (relative
    concentrations) that the percent-variation and rank-sum statistics are computed
    from
  repo: https://github.com/xdomingoal/erah-devel
- name: R
  role: Language in which the Wilcoxon-Mann-Whitney rank-sum test and percent-variation
    arithmetic are executed
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:54e104ec7807a148d90f389621cae1809fe4a10796463da33094b58b91296696
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xdomingoal/erah-devel
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-percent-variation-calculation

## Summary

Computes the relative fold change of eRah-quantified metabolites between two sample classes (e.g. HIAE patients vs healthy controls) as 100*(mean_case-mean_control)/mean_control, then attaches a two-sided Wilcoxon-Mann-Whitney rank-sum p-value per metabolite to reproduce the Table 3-style dysregulation statistics.

## When to use

Use when an eRah deconvolution/alignment run yields relative concentrations of putatively identified compounds for two sample groups (e.g. HIAE vs CTR) and you need per-metabolite percent variation and rank-sum significance to report dysregulated metabolites.

## When NOT to use

- Input is already a processed feature table of fold changes or log2 ratios — percent-variation recomputation would be redundant
- Data are raw GC/MS chromatograms that have not yet been deconvolved and aligned by eRah (deconvolveComp, alignComp, identifyComp must run first)
- More than two sample classes or paired/replicate designs that violate the two-sample Wilcoxon-Mann-Whitney assumption

## Inputs

- eRah output table of relative metabolite concentrations per sample (compound names, matching scores, integrated area), e.g. Supporting File 4 covering the 25 serum samples
- Sample-class labels assigning each sample to HIAE (n=11) or CTR (n=14)

## Outputs

- CSV table (e.g. table3_metabolite_stats.csv) with columns: metabolite name, percent variation (100*(mean_case-mean_control)/mean_control), two-sided Wilcoxon-Mann-Whitney p-value

## How to apply

Load the eRah output table (compound names, matching scores and integrated area per sample; e.g. Supporting File 4 with relative concentrations for the 25 serum samples) and subset the rows of interest (e.g. lactic acid, leucine 1TMS, leucine 2TMS, serine, 5-oxoproline, glutamic acid, ornithine, lysine). Split the relative concentrations by class label (11 HIAE patients vs 14 CTR controls) and compute 100*(mean(HIAE)-mean(CTR))/mean(CTR) for each metabolite. In R, run a two-sided Wilcoxon-Mann-Whitney rank-sum test per metabolite comparing the two groups. Assemble metabolite name, percent variation and p-value into a single table (e.g. table3_metabolite_stats.csv). The article reports these metabolites as significantly elevated with p-value < 0.01, which is the significance threshold to check against.

## Related tools

- **erah** (Produces the deconvolved, aligned and identified compound table (relative concentrations) that the percent-variation and rank-sum statistics are computed from) — https://github.com/xdomingoal/erah-devel
- **R** (Language in which the Wilcoxon-Mann-Whitney rank-sum test and percent-variation arithmetic are executed) — http://CRAN.R-project.org/package=erah

## Examples

```
wilcox.test(hiae, ctr, alternative='two.sided'); pct <- 100*(mean(hiae)-mean(ctr))/mean(ctr)
```

## Evaluation signals

- All eight recomputed p-values fall below the significance threshold (p-value < 0.01 as reported in Table 3)
- Percent-variation values match the direction and magnitude reported in Table 3 for each metabolite
- Group sizes equal 11 (HIAE) and 14 (CTR) after label-based subsetting
- Only the eight named dysregulated metabolites (lactic acid, leucine 1TMS/2TMS, serine, 5-oxoproline, glutamic acid, ornithine, lysine) are listed in the output table
- Percent-variation sign is positive for every metabolite, confirming elevation in HIAE vs CTR

## Limitations

- Validation scope limited to serum samples analyzed by GC–TOF MS
- Supporting methods and data files (Supporting Files 1–4) are hosted externally at pubs.acs.org rather than included in the main text
- No specific version number stated for the eRah package

## Evidence

- [supplementary] table3_metabolite_stats computation: [evidence span withheld — anchor status: not_found]
- [supplementary] statistical test choice: [evidence span withheld — anchor status: not_found]
- [results] significance threshold: [evidence span withheld — anchor status: not_found]
- [abstract] eRah output format: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [results] input data source: [evidence span withheld — anchor status: not_found]
- [discussion] validation scope: "The comparative analysis of serum samples by GC–TOF MS provided test data"
