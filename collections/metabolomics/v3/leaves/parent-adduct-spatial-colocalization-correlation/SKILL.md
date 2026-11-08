---
name: parent-adduct-spatial-colocalization-correlation
description: Use when you have an msimat MSI peak table and a set of candidate parent–adduct
  mass-difference pairs (from diffGetPeaks/adductMatch) and need to filter them by
  pixel-intensity correlation before counting adduct abundance.
when_to_use_negative:
- The input is a plain list of mass values without pixel intensities — correlation
  of ion images is impossible and only mass-difference matching (adductMatch/topAdducts)
  applies.
- The MSI data have not been peak-binned/processed (Cardinal objects must already
  be MSProcessedImagingExperiment/MSContinuousImagingExperiment after peakBin), since
  raw spectra lack a defined peak list to correlate.
- The adduct of interest does not co-localize with its parent (e.g. diffuse matrix
  background) — the Pearson correlation test will reject it and the pair should not
  be counted as a true adduct.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: mass2adduct
  role: Provides corrPairsMSI/corrPairsMSIchunks for the spatial Pearson correlation
    test between parent and putative adduct ion images in msimat format
  repo: https://github.com/kbseah/mass2adduct
- name: Cardinal
  role: Used to read imzML and pre-process (normalize, smoothSignal, reduceBaseline,
    peakBin) MSI data before conversion to msimat for correlation analysis
  repo: http://cardinalmsi.org/
- name: MSiReader
  role: Source of plain-text CSV intensity exports that serve as the correlation input
    after msimat import
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:8ee82409eca8db0dded20f31cb0cdd22dcd59324c9bc5e31408e88a8e2eeae44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# parent-adduct-spatial-colocalization-correlation

## Summary

Spatially validates putative metabolite–matrix adduct pairs (e.g. [M + (DHB-H2O) + H]+) by testing whether parent-ion and adduct-ion images in MALDI-MSI data are significantly correlated, so only co-localizing mass-difference pairs are counted as real adducts.

## When to use

Use when you have an msimat MSI peak table and a set of candidate parent–adduct mass-difference pairs (from diffGetPeaks/adductMatch) and need to filter them by pixel-intensity correlation before counting adduct abundance.

## When NOT to use

- The input is a plain list of mass values without pixel intensities — correlation of ion images is impossible and only mass-difference matching (adductMatch/topAdducts) applies.
- The MSI data have not been peak-binned/processed (Cardinal objects must already be MSProcessedImagingExperiment/MSContinuousImagingExperiment after peakBin), since raw spectra lack a defined peak list to correlate.
- The adduct of interest does not co-localize with its parent (e.g. diffuse matrix background) — the Pearson correlation test will reject it and the pair should not be counted as a true adduct.

## Inputs

- msimat object of MSI peak intensities (CSV export from SCiLS or MSiReader, or converted from imzML via Cardinal peakBin + cardinal2msimat)
- massdiff object of candidate parent–adduct mass-difference pairs (e.g. subset via diffGetPeaks at mass=136.01600 for DHB-H2O)

## Outputs

- data.frame of per-pair correlation test results (Pearson p-values, Bonferroni-corrected at cutoff p=0.05)
- List of significantly co-localized parent–adduct ion pairs retained for adduct-abundance counting

## How to apply

First import the MALDI-MSI data as an `msimat` object (CSV export from SCiLS or MSiReader, or `cardinal2msimat` on a `peakBin`-processed Cardinal MSProcessedImagingExperiment). Compute all pairwise mass differences with `massdiff(peaks)` and subset candidate pairs for a specific adduct mass, e.g. `diffGetPeaks(d.diff, mass=136.01600, mDa=1)` for the DHB-H2O adduct (a window of ±0.5 mDa or a ppm-proportional window). Because adduct formation should make parent and derivative ion abundances co-vary across pixels, run `corrPairsMSI(d, d.diff.DHBH2O)` — a two-tailed Pearson correlation test per ion pair with a p=0.05 significance cutoff under Bonferroni correction. For large datasets that exceed RAM, use `corrPairsMSIchunks` with `mem.limit=N` to serially process the correlation in memory-bounded chunks; on Unix, `how="parallel", ncores=4` accelerates it. The output data.frame of p-values determines which putative adduct pairs are retained for downstream adduct-fraction quantification.

## Related tools

- **mass2adduct** (Provides corrPairsMSI/corrPairsMSIchunks for the spatial Pearson correlation test between parent and putative adduct ion images in msimat format) — https://github.com/kbseah/mass2adduct
- **Cardinal** (Used to read imzML and pre-process (normalize, smoothSignal, reduceBaseline, peakBin) MSI data before conversion to msimat for correlation analysis) — http://cardinalmsi.org/
- **MSiReader** (Source of plain-text CSV intensity exports that serve as the correlation input after msimat import)

## Examples

```
d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- Correlation output is a data.frame with p-values for each candidate ion pair, significant at p=0.05 after Bonferroni correction by default.
- Candidate pairs must have been subset from the full massdiff object with diffGetPeaks at a defined window (e.g. mDa=1 around mass 136.01600) — verify the subset matches the intended adduct mass.
- Parent–adduct ion images should visually co-localize; spot-check a few significant pairs by plotting their spatial distributions before counting them as adducts.
- For very large CSV exports, corrPairsMSIchunks must be used with mem.limit set below available RAM; check that chunked results match non-chunked results on a small test subset.
- Downstream adduct-fraction counts should only include pairs that passed the correlation filter — compare the filtered fraction against the reported ~31% (AP-SMALDI/Orbitrap) vs ~6% (high-vacuum) platform contrast if replicating Figure 3.

## Limitations

- Correlation tables for very large MSI datasets can exceed memory, requiring the chunked corrPairsMSIchunks variant; even msimunging.pl triplet-format conversion is still limited by system RAM.
- Pearson correlation assumes co-variation of ion intensities; matrix-derived ions distributed independently of the metabolite may fail the test, while true spatial co-localization is necessary but may not alone prove adduct identity (mass accuracy and matching to the reference adduct table also matter).
- Default significance uses Bonferroni correction, which is conservative for the large numbers of pairwise tests in MSI data.
- Parallel mode is limited to Unix systems (how="parallel").

## Evidence

- [readme] Correlation of parent and adduct ion images: [evidence span withheld — anchor status: external_doc]
- [readme] Rationale for co-localization test: [evidence span withheld — anchor status: external_doc]
- [readme] Correlation function invocation: [evidence span withheld — anchor status: external_doc]
- [readme] Significance cutoff and correction: [evidence span withheld — anchor status: external_doc]
- [readme] Memory-bounded chunking for large datasets: [evidence span withheld — anchor status: external_doc]
- [abstract] Estimation strategy from abstract: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance"
- [methods] Correlation as candidate filter: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
