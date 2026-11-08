---
name: multiple-testing-bonferroni-correction
description: Use when calling significant parent–adduct ion pairs from massdiff/corrPairsMSI
  output — i.e. when each candidate pair's Pearson p-value must be judged against
  a family-wise threshold to avoid false adduct calls.
when_to_use_negative:
- Input is already a filtered list of few ion pairs where the multiplicity correction
  would be overly conservative (Bonferroni's family-wise threshold assumes many simultaneous
  tests).
- The analysis goal is adduct discovery by mass-difference histogram matching (adductMatch/topAdducts),
  not statistical validation of spatial co-localization.
- Testing non-pairwise metabolite correlations where a less conservative FDR-based
  approach is expected rather than family-wise error control.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: mass2adduct
  role: Implements corrPairsMSI, which performs the Pearson correlation test on each
    parent–adduct pair and applies Bonferroni correction (default p=0.05 cutoff) to
    flag significant pairs.
  repo: https://github.com/kbseah/mass2adduct
- name: mass2adduct (corrPairsMSIchunks)
  role: Memory-safe variant for very large MSI datasets, breaking the correlation+correction
    problem into chunks processed serially so the same Bonferroni calls can be made
    without loading everything into RAM.
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment in which the mass2adduct correlation and correction functions
    are executed.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:d46d532c39d719cedfb8d7a7a34e21fe0bd2277e3533286f81d201ac32c8831a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - R
  - mass2adduct
  - mass2adduct (corrPairsMSIchunks)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multiple-testing-bonferroni-correction

## Summary

Bonferroni correction applied within mass2adduct to control false positives when testing spatial correlations of putative parent–adduct ion pairs in MALDI-MSI peak-intensity data. It sets a family-wise significance threshold over the many ion pairs tested, so only confidently co-localized adduct pairs are retained.

## When to use

Use when calling significant parent–adduct ion pairs from massdiff/corrPairsMSI output — i.e. when each candidate pair's Pearson p-value must be judged against a family-wise threshold to avoid false adduct calls.

## When NOT to use

- Input is already a filtered list of few ion pairs where the multiplicity correction would be overly conservative (Bonferroni's family-wise threshold assumes many simultaneous tests).
- The analysis goal is adduct discovery by mass-difference histogram matching (adductMatch/topAdducts), not statistical validation of spatial co-localization.
- Testing non-pairwise metabolite correlations where a less conservative FDR-based approach is expected rather than family-wise error control.

## Inputs

- msimat object built from a MALDI-MSI peak-intensity CSV exported from SCiLS or MSiReader (or converted from a Cardinal MSProcessedImagingExperiment via cardinal2msimat)
- massdiff object of candidate parent–adduct mass pairs (e.g. from diffGetPeaks with mass=136.01600, mDa=1)
- optional: list of candidate pairs for chunked processing when the correlation table exceeds available memory

## Outputs

- data.frame of ion pairs with Estimate (Pearson correlation coefficient), P.value, and Significance column indicating Bonferroni-corrected significance
- filtered adduct-pair table (e.g. adduct_pair_table.csv) retaining only significant parent–adduct pairs

## How to apply

Run corrPairsMSI on the msimat peak-intensity object together with the candidate massdiff pairs (e.g. those matching DHB-H2O at 136.01600 within a 1 mDa window) to obtain a two-tailed Pearson p-value per ion pair. Because thousands of pairs are tested, apply Bonferroni correction: the output data.frame flags each pair's P.value against a cutoff computed from the default 0.05 alpha divided by the number of tests, and marks the pair as significant if P.value falls below that threshold. The Significance column lets you filter the matched adduct-pair table to keep only parent–adduct pairs whose spatial co-occurrence is unlikely by chance, which is the evidence that true adduct formation (not random overlap) produced the mass difference. For datasets too large for memory, run corrPairsMSIchunks with a specified mem.limit so the same Bonferroni-corrected calls can be made chunk-wise. Save the filtered result as adduct_pair_table.csv, retaining Estimate (correlation coefficient), P.value, and Significance columns for each pair.

## Related tools

- **mass2adduct** (Implements corrPairsMSI, which performs the Pearson correlation test on each parent–adduct pair and applies Bonferroni correction (default p=0.05 cutoff) to flag significant pairs.) — https://github.com/kbseah/mass2adduct
- **mass2adduct (corrPairsMSIchunks)** (Memory-safe variant for very large MSI datasets, breaking the correlation+correction problem into chunks processed serially so the same Bonferroni calls can be made without loading everything into RAM.) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment in which the mass2adduct correlation and correction functions are executed.)

## Examples

```
d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O, how="parallel", ncores=4)
```

## Evaluation signals

- Every reported ion pair has non-missing Estimate, P.value, and Significance columns; Significance is computed against the Bonferroni-corrected 0.05/N cutoff, not raw p<0.05.
- Significant pairs' P.value values are all below the Bonferroni threshold implied by the number of tests; Estimate (Pearson r) for accepted pairs should be near +1 for true adduct formation.
- Matched adduct-pair table retains only pairs whose mass difference was previously annotated by adductMatch/diffGetPeaks (e.g. DHB-H2O ~136.01600 within 1 mDa) — the correction step should not invent pairs absent from the massdiff object.
- When run via corrPairsMSIchunks with a defined mem.limit, the summed chunk-wise significant set matches the result from the serial corrPairsMSI run on the same pairs.
- Output adduct_pair_table.csv is a data.frame whose row count equals the number of candidate pairs fed in, with no rows dropped due to missing correlation values.

## Limitations

- Bonferroni correction is conservative; it may miss truly additive adduct pairs when many tests are performed, so co-localization-based calls can be incomplete.
- Pearson correlation assumes roughly linear co-occurrence of parent and adduct intensities across pixels; nonlinear or negative correlations are not captured by this skill.
- The corrected significance reflects spatial co-occurrence, not chemical identity — a significant pair still needs the adductMatch annotation to be a credible metabolite–matrix adduct (e.g. DHB-H2O, DHB+Na).
- Memory constraints on large plain-text CSV MSI exports (on the order of several Gb) can force the use of corrPairsMSIchunks; the default corrPairsMSI will fail if the correlation table does not fit in RAM.

## Evidence

- [methods] Bonferroni correction on mass2adduct correlation tests: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
