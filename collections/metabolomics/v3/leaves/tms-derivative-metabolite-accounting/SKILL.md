---
name: tms-derivative-metabolite-accounting
description: Use when recomputing or verifying dysregulated-metabolite statistics
  from an eRah compound table in which one metabolite occurs in several TMS-derivatized
  rows (e.g., leucine 1TMS/2TMS) and two-group class comparisons are required.
when_to_use_negative:
- Input is raw GC-TOF MS profile data rather than an eRah-aligned identification table
  — deconvolution and alignment (deconvolveComp, alignComp) must be run first before
  metabolite-level accounting is possible
- The comparison involves more than two sample classes — the Wilcoxon-Mann-Whitney
  rank-sum test used in this workflow is defined only for a two-group HIAE vs CTR
  comparison
- The metabolite table lacks TMS-derivative annotations, so derivatized rows cannot
  be attributed to a parent metabolite and derivative-level accounting cannot be performed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: erah
  role: Produces the compound table (compound names, matching scores, integrated area
    per sample) from which the TMS-derivative metabolite rows and relative concentrations
    for the 25 serum samples are taken
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Environment in which the two-sided Wilcoxon-Mann-Whitney rank-sum p-values
    are computed per metabolite and the result table is assembled
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:222af7797c30bb5536702da2eac94604d752e6bd0257671dead338add8af7320
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tms-derivative-metabolite-accounting

## Summary

Correctly accounts for GC/MS metabolites that appear as multiple TMS-derivative rows (e.g., leucine 1TMS and leucine 2TMS) when recomputing percent-variation and Wilcoxon-Mann-Whitney statistics for dysregulated metabolites identified by eRah in the HIAE vs. control serum comparison.

## When to use

Use when recomputing or verifying dysregulated-metabolite statistics from an eRah compound table in which one metabolite occurs in several TMS-derivatized rows (e.g., leucine 1TMS/2TMS) and two-group class comparisons are required.

## When NOT to use

- Input is raw GC-TOF MS profile data rather than an eRah-aligned identification table — deconvolution and alignment (deconvolveComp, alignComp) must be run first before metabolite-level accounting is possible
- The comparison involves more than two sample classes — the Wilcoxon-Mann-Whitney rank-sum test used in this workflow is defined only for a two-group HIAE vs CTR comparison
- The metabolite table lacks TMS-derivative annotations, so derivatized rows cannot be attributed to a parent metabolite and derivative-level accounting cannot be performed

## Inputs

- Supporting File 4: eRah putative identifications with relative concentrations for the 25 serum samples (compound names, matching scores, integrated areas per sample)
- Class label vector assigning each of the 25 samples to HIAE patient (n=11) or CTR control (n=14)
- List of target metabolite names with their TMS-derivative row variants (e.g., leucine 1TMS, leucine 2TMS)

## Outputs

- table3_metabolite_stats.csv: one row per metabolite with metabolite name, percentage of variation [100*(mean(HIAE)-mean(CTR))/mean(CTR)] and two-sided Wilcoxon-Mann-Whitney rank-sum p-value

## How to apply

Load Supporting File 4, the eRah putative identifications with relative concentrations for the 25 serum samples, and subset the rows for the target metabolites, treating derivatized forms such as leucine 1TMS and leucine 2TMS as rows of the same parent metabolite so they are not double-counted. Split the relative concentrations by class label into 11 HIAE patients and 14 CTR controls. For each metabolite compute the percentage of variation as 100*(mean(HIAE)-mean(CTR))/mean(CTR). In R, compute a two-sided Wilcoxon-Mann-Whitney rank-sum p-value per metabolite comparing HIAE vs CTR. Assemble metabolite name, percent variation and p-value into one table saved as table3_metabolite_stats.csv, which should reproduce the direction and magnitude of the Table 3 results where all eight metabolites (lactic acid, leucine, serine, 5-oxoproline, glutamic acid, ornithine, lysine) were significantly elevated with p-value < 0.01.

## Related tools

- **erah** (Produces the compound table (compound names, matching scores, integrated area per sample) from which the TMS-derivative metabolite rows and relative concentrations for the 25 serum samples are taken) — http://CRAN.R-project.org/package=erah
- **R** (Environment in which the two-sided Wilcoxon-Mann-Whitney rank-sum p-values are computed per metabolite and the result table is assembled)

## Examples

```
wilcox.test(conc[class=='HIAE'], conc[class=='CTR'], alternative='two.sided')
```

## Evaluation signals

- All eight recomputed p-values fall below the significance threshold (p < 0.01, as reported in Table 3)
- Percent-variation values match the direction and magnitude of those reported in Table 3 of the paper
- Sample grouping is complete: 11 HIAE patients plus 14 CTR controls equals the 25 serum samples in Supporting File 4, with no samples dropped or duplicated
- Derivatized rows such as leucine 1TMS and leucine 2TMS are mapped to the single parent metabolite leucine without double-counting in the final statistics table
- The output table has exactly the three expected columns (metabolite name, percent variation, p-value) and all seven metabolites were elevated in HIAE relative to CTR

## Limitations

- Supporting methods and data files (Supporting Files 1-4) are hosted externally at pubs.acs.org rather than included in the main text, so the derivative-annotated input must be retrieved separately
- No specific version number is stated for the eRah package, so recomputation may depend on the eRah version used to generate the identifications
- Validation scope is limited to the serum samples analyzed by GC-TOF MS in the HIAE vs control comparison

## Evidence

- [readme] eRah outputs a table with compound names, matching scores and the integrated area of the compound for each sample.: "Outputs a table with compound names, matching scores and the integrated area of the compound for each sample."
- [intro] We demonstrate the functionality of e Rah through a comparative analysis of serum samples from adolescents with hyperinsulinaemic androgen excess: "We demonstrate the functionality of e Rah through a comparative analysis of serum samples from adolescents with hyperinsulinaemic androgen excess"
- [abstract] Significantly dysregulated metabolites are further validated using pure standards and targeted analysis by GC-triple quadrupole (QqQ) MS, LC-QqQ, and NMR.: "Significantly dysregulated metabolites are further validated using pure standards and targeted analysis by GC-triple quadrupole (QqQ) MS, LC-QqQ, and NMR."
- [methods] Serum aliquots (25 µL) were thawed at 4 ◦C. Samples were briefly vortex-mixed and each aliquot was supplemented with 20 µL of 1 µg/µL succinic-d4 acid: "Serum aliquots (25 µL) were thawed at 4 ◦C. Samples were briefly vortex-mixed and each aliquot was supplemented with 20 µL of 1 µg/µL succinic-d4 acid"
