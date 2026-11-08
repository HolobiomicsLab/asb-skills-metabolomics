---
name: multipletesting-bonferroni-correction
description: Use when many spatial correlation tests (e.g. Pearson tests over all
  parent–adduct ion pairs from massdiff objects) are run simultaneously and per-pair
  p-values must be thresholded to avoid false-positive adduct assignments.
when_to_use_negative:
- When only a handful of pre-specified hypotheses are tested (single ion pair per
  adduct) — the correction is trivial and the paired-test workflow is unnecessary
  overhead.
- When the goal is exploratory ranking of mass-difference histogram peaks (histWeighted,
  topAdducts, adductMatch) rather than confirmatory significance testing — those steps
  do not use p-values.
- When raw p-values are needed for a different multiple-testing policy (e.g. FDR control);
  corrPairsMSI's Significance column is hard-wired to Bonferroni at p=0.05.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mass2adduct
  role: R package whose corrPairsMSI/corrPairsMSIchunks functions run the per-pair
    Pearson tests and apply the Bonferroni-corrected significance threshold
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: execution environment for the mass2adduct workflow (library(mass2adduct))
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:9d24c9cb70c391e6bbf64e4ce906e62333cbd8ade7d50724df9d6f5cecfd3543
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - R
  - mass2adduct
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multipletesting-bonferroni-correction

## Summary

Apply Bonferroni correction to control the family-wise error rate when testing thousands of parent–adduct ion pairs for spatial correlation in MALDI-MSI data, so that adduct calls in mass2adduct's massdiff output are significant at p=0.05 after correction.

## When to use

Use when many spatial correlation tests (e.g. Pearson tests over all parent–adduct ion pairs from massdiff objects) are run simultaneously and per-pair p-values must be thresholded to avoid false-positive adduct assignments.

## When NOT to use

- When only a handful of pre-specified hypotheses are tested (single ion pair per adduct) — the correction is trivial and the paired-test workflow is unnecessary overhead.
- When the goal is exploratory ranking of mass-difference histogram peaks (histWeighted, topAdducts, adductMatch) rather than confirmatory significance testing — those steps do not use p-values.
- When raw p-values are needed for a different multiple-testing policy (e.g. FDR control); corrPairsMSI's Significance column is hard-wired to Bonferroni at p=0.05.

## Inputs

- msimat object (d) holding pixel-by-peak MSI intensities imported from a semicolon-separated CSV exported by SCiLS or MSiReader
- massdiff object of parent–adduct ion pairs (d.diff or d.diff.annot), e.g. filtered with diffGetPeaks(mass=136.01600, mDa=1) for the DHB-H2O adduct

## Outputs

- data.frame (annotated massdiff object) with Estimate, P.value, and Significance columns per ion pair, where Significance encodes the Bonferroni-corrected p=0.05 decision
- annotated_massdiff_corr.csv containing correlation results for downstream adduct counting

## How to apply

Run corrPairsMSI(d, d.diff.annot) on the msimat object and the annotated massdiff object: by default it performs a two-tailed Pearson correlation test on each parent–adduct peak pair and appends Estimate, P.value, and Significance columns. The Significance column applies the Bonferroni correction by comparing each raw P.value against a family-wise threshold of p=0.05 divided by the number of ion pairs tested. Only ion pairs flagged significant in this column should be counted as true metabolite–matrix adducts (e.g. [M + (DHB-H2O) + H]+ or [M + DHB + Na]+ with DHB matrix). Rationale: with thousands of pairwise tests, uncorrected p=0.05 would yield many spurious adduct assignments; for very large datasets use corrPairsMSIchunks with mem.limit=N, which preserves the same per-pair testing and correction scheme while processing serially in chunks.

## Related tools

- **mass2adduct** (R package whose corrPairsMSI/corrPairsMSIchunks functions run the per-pair Pearson tests and apply the Bonferroni-corrected significance threshold) — https://github.com/kbseah/mass2adduct
- **R** (execution environment for the mass2adduct workflow (library(mass2adduct)))

## Examples

```
d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O, how="parallel", ncores=4)
```

## Evaluation signals

- Output data.frame contains exactly the columns Estimate, P.value, and Significance for every ion pair in the input massdiff object.
- Number of tested pairs equals the number of rows in the massdiff/diffGetPeaks subset; no pairs silently dropped.
- Every pair flagged in Significance has a raw P.value below 0.05 divided by the number of tested pairs (Bonferroni threshold).
- Known true adducts (e.g. DHB-H2O pairs from the 7 adduct-forming amine-containing metabolite standards) come out significant, while random unrelated peak pairs do not.
- corrPairsMSIchunks with mem.limit=N yields the same Significance calls as corrPairsMSI on the same input.

## Limitations

- Bonferroni at family-wise p=0.05 is conservative; weak but real adduct co-localizations may fail the corrected threshold.
- corrPairsMSI loads all correlations into memory; very large MSI tables may not fit and require corrPairsMSIchunks (still memory-limited, e.g. a 40 Gb CSV with 32 Gb RAM will probably not work).
- Correlation significance does not by itself prove chemical adduction — putative pairs must first be matched to known adduct masses (adducts2 reference table) within the mass tolerance of the instrument.

## Evidence

- [methods] Correlation test with Bonferroni correction in mass2adduct: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] Significance threshold definition: [evidence span withheld — anchor status: external_doc]
- [other] Output columns from the task workflow: [evidence span withheld — anchor status: not_found]
- [readme] Chunked alternative preserves the test scheme: "To avoid this, use `corrPairsMSIchunks` instead of `corrPairsMSI`, and specify the maximum available memory with `mem.limit=N`"
