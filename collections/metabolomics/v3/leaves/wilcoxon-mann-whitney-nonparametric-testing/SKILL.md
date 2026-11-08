---
name: wilcoxon-mann-whitney-nonparametric-testing
description: Use when comparing GC–MS metabolite relative concentrations between two
  small, independent sample groups (e.g. 11 disease vs 14 control serum samples) and
  a nonparametric test is needed to call metabolites significantly dysregulated.
when_to_use_negative:
- When the eRah deconvolution/alignment/identification pipeline has not yet produced
  per-sample relative concentrations for the serum samples
- When comparing more than two groups or paired samples, where the two-sample Wilcoxon-Mann-Whitney
  rank-sum design does not apply
- When the concentration table has not been filtered to the specific dysregulated
  metabolites and class labels are missing
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: erah
  role: Produces the per-sample relative-concentration table (compound names, matching
    scores, integrated area) that serves as the statistical input
  repo: https://github.com/xdomingoal/erah-devel
- name: R
  role: Language in which the two-sided Wilcoxon-Mann-Whitney rank-sum p-values are
    computed and the stats table is assembled
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:881d7ccb7e89eef94901a927d2e9efa43194a7b00b7afb009f507ad76dff0124
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xdomingoal/erah-devel
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# wilcoxon-mann-whitney-nonparametric-testing

## Summary

Apply two-sided Wilcoxon-Mann-Whitney rank-sum tests in R to serum relative concentrations (e.g. the 11 HIAE vs 14 CTR samples in the eRah GC–MS comparative analysis) to flag significantly dysregulated metabolites such as the eight reported in Table 3.

## When to use

Use when comparing GC–MS metabolite relative concentrations between two small, independent sample groups (e.g. 11 disease vs 14 control serum samples) and a nonparametric test is needed to call metabolites significantly dysregulated.

## When NOT to use

- When the eRah deconvolution/alignment/identification pipeline has not yet produced per-sample relative concentrations for the serum samples
- When comparing more than two groups or paired samples, where the two-sample Wilcoxon-Mann-Whitney rank-sum design does not apply
- When the concentration table has not been filtered to the specific dysregulated metabolites and class labels are missing

## Inputs

- Metabolite relative-concentration table from eRah putative identifications (e.g. Supporting File 4 for the 25 serum samples)
- Sample class-label vector distinguishing the two groups (11 HIAE patients vs 14 CTR controls)
- List of metabolite rows to test (e.g. lactic acid, leucine, serine, 5-oxoproline, glutamic acid, ornithine, lysine)

## Outputs

- Per-metabolite table of percentage of variation (100*(mean(HIAE)-mean(CTR))/mean(CTR)) and two-sided Wilcoxon-Mann-Whitney rank-sum p-value (e.g. table3_metabolite_stats.csv)

## How to apply

Load the metabolite relative-concentration table produced by eRah (e.g. Supporting File 4, the eRah putative identifications with relative concentrations for the 25 serum samples) and subset rows for the metabolites of interest, such as lactic acid, leucine 1TMS, leucine 2TMS, serine, 5-oxoproline, glutamic acid, ornithine and lysine. Split relative concentrations by class label, here 11 hyperinsulinaemic androgen excess (HIAE) patients versus 14 healthy controls (CTR). For each metabolite compute the percentage of variation as 100*(mean(HIAE)-mean(CTR))/mean(CTR) and, in R, compute a two-sided Wilcoxon-Mann-Whitney rank-sum p-value comparing the two groups. Assemble metabolite name, percent variation and p-value into a single table (e.g. table3_metabolite_stats.csv). Because eRah's deconvolution and alignment are the upstream steps, validate that the recomputed p-values fall below the significance threshold and that percent-variation directions and magnitudes match the published Table 3 values.

## Related tools

- **erah** (Produces the per-sample relative-concentration table (compound names, matching scores, integrated area) that serves as the statistical input) — https://github.com/xdomingoal/erah-devel
- **R** (Language in which the two-sided Wilcoxon-Mann-Whitney rank-sum p-values are computed and the stats table is assembled) — http://CRAN.R-project.org/package=erah

## Examples

```
wilcox.test(hiae_conc, ctr_conc, alternative = "two.sided")  # per metabolite row of Supporting File 4
```

## Evaluation signals

- All eight recomputed p-values (lactic acid, leucine 1TMS, leucine 2TMS, serine, 5-oxoproline, glutamic acid, ornithine, lysine) fall below the significance threshold reported in Table 3 (p-value<0.01)
- Percent-variation values match the direction and magnitude of those reported in Table 3 (all eight metabolites elevated in HIAE versus control serum)
- Output table contains exactly the three columns: metabolite name, percent variation, p-value, with 8 rows for the tested metabolites
- Group sizes used are 11 HIAE and 14 CTR samples, matching the 25-sample serum cohort in Supporting File 4

## Limitations

- Validation scope limited to serum samples analyzed by GC–TOF MS; results do not automatically generalize to other matrices
- Supporting methods and data files (Supporting Files 1–4) are hosted externally at pubs.acs.org rather than included in the main text, so reproducibility depends on obtaining those files
- The upstream eRah pipeline (deconvolution, alignment, identification) must be run first; this skill only performs the statistical comparison on its output

## Evidence

- [abstract] Significantly dysregulated metabolites are further validated using pure standards and targeted analysis by GC-triple quadrupole (QqQ) MS, LC-QqQ, and NMR.: "Significantly dysregulated metabolites are further validated using pure standards and targeted analysis by GC-triple quadrupole (QqQ) MS, LC-QqQ, and NMR."
- [results] Lactic acid, leucine, serine, 5-oxoproline (pyroglutamic acid), glutamic acid, ornithine and lysine were significantly elevated in HIAE versus control serum, all with p-value<0.01, as reported in Table 3.: [evidence span withheld — anchor status: not_found]
- [methods] Serum aliquots (25 µL) were thawed at 4 ◦C. Samples were briefly vortex-mixed and each aliquot was supplemented with 20 µL of 1 µg/µL succinic-d4 acid: "Serum aliquots (25 µL) were thawed at 4 ◦C. Samples were briefly vortex-mixed and each aliquot was supplemented with 20 µL of 1 µg/µL succinic-d4 acid"
- [abstract] e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
