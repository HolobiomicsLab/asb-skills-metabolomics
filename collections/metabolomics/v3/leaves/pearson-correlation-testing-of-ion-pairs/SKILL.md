---
name: pearson-correlation-testing-of-ion-pairs
description: Use when an annotated massdiff object of matched adduct ion pairs has
  been generated from MSI peak data and the candidate parent–adduct pairs must be
  tested for spatial co-localization before being accepted as true adducts.
when_to_use_negative:
- The data are not MS imaging data with pixel-level intensity co-ordinates — spatial
  correlation is meaningless without spatial context (e.g. a plain list of masses
  imported via scan() lacks the intensity matrix needed by corrPairsMSI).
- 'You are still at the histogram/annotation stage: if the mass differences have not
  been binned and matched to known adducts (adductMatch), there is no set of candidate
  pairs to test.'
- The MSI intensity table exceeds available memory and has not been prepared for chunked
  processing — running plain corrPairsMSI will fail; use corrPairsMSIchunks or pre-filter/convert
  the CSV with msimunging.pl.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mass2adduct
  role: R package providing corrPairsMSI, corrPairsMSIchunks, msimat, adductMatch
    and diffGetPeaks for testing spatial correlations between candidate parent–adduct
    ion pairs
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: runtime environment in which mass2adduct's correlation functions are executed
- name: Cardinal
  role: source of peak-binned MSI data that can be converted to msimat format via
    cardinal2msimat() before correlation testing
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:5239e7944db7e5f660d97482d4b55a342c91e8420b038817c8b2ba983616a77a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - R
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pearson-correlation-testing-of-ion-pairs

## Summary

Statistical validation of putative metabolite–matrix adduct pairs in MALDI-MSI data by running two-tailed Pearson correlation tests on the pixel intensities of each parent–adduct ion pair, using mass2adduct's corrPairsMSI/corrPairsMSIchunks with Bonferroni-corrected significance at p=0.05.

## When to use

Use when an annotated massdiff object of matched adduct ion pairs has been generated from MSI peak data and the candidate parent–adduct pairs must be tested for spatial co-localization before being accepted as true adducts.

## When NOT to use

- The data are not MS imaging data with pixel-level intensity co-ordinates — spatial correlation is meaningless without spatial context (e.g. a plain list of masses imported via scan() lacks the intensity matrix needed by corrPairsMSI).
- You are still at the histogram/annotation stage: if the mass differences have not been binned and matched to known adducts (adductMatch), there is no set of candidate pairs to test.
- The MSI intensity table exceeds available memory and has not been prepared for chunked processing — running plain corrPairsMSI will fail; use corrPairsMSIchunks or pre-filter/convert the CSV with msimunging.pl.

## Inputs

- msimat object holding MSI peak-intensity values (e.g. from a semicolon-separated CSV exported by SCiLS or MSiReader, or converted from a peak-binned Cardinal MSProcessedImagingExperiment/MSContinuousImagingExperiment via cardinal2msimat())
- Annotated massdiff object of matched parent–adduct ion pairs (output of adductMatch or diffGetPeaks), e.g. a DHB–H2O adduct at mass 136.01600 with a ±1 mDa or ppm window

## Outputs

- data.frame (massdiff object) with columns Estimate, P.value, and Significance (Bonferroni-corrected p-value threshold) for each tested parent–adduct ion pair
- Filtered table of significant parent–adduct pairs (e.g. adduct_pair_table.csv)

## How to apply

Run corrPairsMSI(d, d.diff.annot) from the mass2adduct R package, passing the msimat object holding peak intensities and the annotated massdiff object produced by adductMatch(); the function performs a two-tailed Pearson correlation test on each parent–adduct pair, appending Estimate (correlation coefficient), P.value, and Significance columns. On Unix systems the computation can be parallelized with how="parallel" and ncores=N. Significance is judged against a Bonferroni-corrected threshold at p=0.05; filter the resulting table to pairs whose P.value falls below that cutoff. Rationale: if two ions truly differ by a molecular adduct, their spatial abundances across pixels should be correlated, so only co-localized pairs are retained as credible adduct assignments. For MSI tables too large for memory, substitute corrPairsMSIchunks with mem.limit=N, which processes the correlation tests in serial chunks; very large CSV inputs can additionally be converted to triplet format beforehand with the inst/msimunging.pl script.

## Related tools

- **mass2adduct** (R package providing corrPairsMSI, corrPairsMSIchunks, msimat, adductMatch and diffGetPeaks for testing spatial correlations between candidate parent–adduct ion pairs) — https://github.com/kbseah/mass2adduct
- **R** (runtime environment in which mass2adduct's correlation functions are executed)
- **Cardinal** (source of peak-binned MSI data that can be converted to msimat format via cardinal2msimat() before correlation testing) — http://cardinalmsi.org/

## Examples

```
R: d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O, how="parallel", ncores=4)
```

## Evaluation signals

- Output data.frame contains Estimate, P.value, and Significance columns for every tested ion pair.
- P.value threshold is Bonferroni-corrected at 0.05; significant pairs must fall below this cutoff.
- Number of tested pairs equals the number of annotated parent–adduct pairs in the input massdiff object (no silent drop).
- Estimated correlation coefficients are plausibly high (co-localization) for confirmed adducts such as metabolite–DHB species [M + (DHB-H2O) + H]+ and [M + DHB + Na]+.

## Limitations

- Correlation alone does not prove adduct formation — it is a rapid screening approach whose significance is only as good as the adductMatch annotation and mass tolerance used.
- Large MSI tables (several Gb CSV) may not fit in memory; corrPairsMSIchunks with mem.limit=N must be used, and even msimunging.pl is limited by the machine's RAM.
- Quantile/match values in upstream histogram annotation are usually high because most mass differences have few counts, so the correlation step filters many false positives but depends on the chosen bin width (e.g. 0.01 in the vignette).

## Evidence

- [readme] Correlation test rationale: [evidence span withheld — anchor status: external_doc]
- [methods] Correlation test mechanics: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] Chunked execution for large data: "Use the function `corrPairsMSIchunks` instead of `corrPairsMSI`, and specify the maximum available memory with `mem.limit=N`"
- [readme] Bonferroni significance default: [evidence span withheld — anchor status: external_doc]
- [discussion] Screening rationale: "perform a simple mass difference calculation and spatial correlation analysis as a rapid and efficient way to screen for these adducts in existing MSI datasets"
