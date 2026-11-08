---
name: mummichog-pathway-enrichment-analysis
description: Use when you have LC-MS1 feature tables (from raw mzML processing) and/or
  MS2-based compound identifications and need pathway enrichment/functional interpretation
  of untargeted metabolomics data.
when_to_use_negative:
- Input is already a fully identified and curated compound list with quantitative
  values — direct statistical/enrichment analysis on named metabolites is more appropriate
- Data are not LC-MS-based (e.g., NMR or MS2-only spectra without MS1 feature context),
  since mummichog operates on MS1 m/z features and adduct patterns
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalystR 4.0
  role: R package providing Convert2Mummichog and the improved/debiased mummichog
    functional interpretation module, plus upstream MS1 processing and MS2 deconvolution/identification
    functions
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the unified LC-MS workflow and mummichog
    functional analysis are executed
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Underlying auto-optimized LC-MS1 feature detection/quantification engine producing
    the feature table used as the mummichog/MS2 target list
  repo: https://github.com/xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:2618e0592ef073905ea15fc7e8eef78ec684c0314f58956aedade18768a39fd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mummichog-pathway-enrichment-analysis

## Summary

Run the improved mummichog pathway/functional enrichment analysis from MetaboAnalystR 4.0 on LC-MS1 feature tables, optionally debiased by MS2-based compound identifications, to detect significantly enriched metabolite pathways directly from peaks without requiring full compound-level annotation.

## When to use

Use when you have LC-MS1 feature tables (from raw mzML processing) and/or MS2-based compound identifications and need pathway enrichment/functional interpretation of untargeted metabolomics data.

## When NOT to use

- Input is already a fully identified and curated compound list with quantitative values — direct statistical/enrichment analysis on named metabolites is more appropriate
- Data are not LC-MS-based (e.g., NMR or MS2-only spectra without MS1 feature context), since mummichog operates on MS1 m/z features and adduct patterns

## Inputs

- Raw centroided LC-MS mzML files (e.g., Metabolomics Workbench ST002796, ST002798)
- MS1 feature table with m/z, retention time, and intensities from auto-optimized peak picking/alignment/gap filling
- MS2-based compound identification results (data frame from PerformResultsExport, from DDA or SWATH-DIA deconvolution)

## Outputs

- Pathway enrichment results table (e.g., .csv) listing significant pathways for MS1-only and MS2-filtered runs
- Side-by-side comparison of enriched pathways (e.g., Fig. 5c-e style) distinguishing unfiltered vs MS2-filtered analyses

## How to apply

Process raw centroided mzML files with MetaboAnalystR 4.0's auto-optimized LC-MS1 workflow (peak picking, alignment, gap filling, annotation) to produce a complete feature table. If MS2 data are available, deconvolve with PerformDDADeconvolution (DDA) or PerformDIADeconvolution (SWATH-DIA), build consensus spectra with PerformSpectrumConsenus using a 50% fragment frequency threshold, and identify compounds with PerformDBSearchingBatch against the curated SQLite reference libraries, exporting results via PerformResultsExport. Then call Convert2Mummichog twice for a controlled comparison: once on MS1 features alone, and once using MS2-based compound identifications to filter out unfeasible empirical-compound candidates before the permutation test. The rationale is that mummichog infers pathway activity from m/z values and adducts without needing definitive annotations, but 'traditional functional enrichment analysis relies on accurate compound identification', so MS2 filtering improves sensitivity and debiases the test — e.g., recovering four additional significant pathways (phosphatidylinositol phosphate metabolism, vitamin D, vitamin C, arachidonic cascade) on the SWATH-DIA COVID-19 dataset.

## Related tools

- **MetaboAnalystR 4.0** (R package providing Convert2Mummichog and the improved/debiased mummichog functional interpretation module, plus upstream MS1 processing and MS2 deconvolution/identification functions) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the unified LC-MS workflow and mummichog functional analysis are executed) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (Underlying auto-optimized LC-MS1 feature detection/quantification engine producing the feature table used as the mummichog/MS2 target list) — https://github.com/xia-lab/OptiLCMS

## Examples

```
mSet <- Convert2Mummichog(mSet)
```

## Evaluation signals

- Output enrichment table exists as a .csv and contains the four additional significant pathways (phosphatidylinositol phosphate metabolism, vitamin D, vitamin C metabolism, arachidonic cascade) for the MS2-filtered SWATH-DIA run
- MS1-only and MS2-filtered runs are clearly distinguishable in the compiled comparison, with the filtered run showing equal or greater pathway coverage
- The mummichog run consumed a complete MS1 feature table (post peak picking, alignment, gap filling) rather than partially annotated peaks
- For DIA data, consensus spectra were built with the 50% fragment frequency threshold before database searching, so identifications feeding the filter are reproducible across replicates
- Empirical-compound candidates ruled infeasible by MS2 identifications are absent from the filtered permutation-test input

## Limitations

- MS2-based compound identification depends on reference spectra databases that are incomplete, so filtering can only act on compounds present in the ~1.5 million-spectra knowledgebases
- Built-in MS2 reference spectra are compiled from various instruments and experimental conditions with potentially varying qualities
- The mummichog permutation test and enrichment are only as good as the upstream deconvolution — chimeric DDA spectra (>half of spectra) and DIA precursor-fragment relinking errors propagate into identifications

## Evidence

- [abstract] more accurate functional interpretation through integrated spectral annotation: "more accurate functional interpretation through integrated spectral annotation"
- [intro] Traditional functional enrichment analysis relies on accurate compound identification.: "Traditional functional enrichment analysis relies on accurate compound identification."
- [intro] The performance could be further improved by utilizing MS2 spectral data.: "The performance could be further improved by utilizing MS2 spectral data."
- [methods] The database search results can be exported as a data frame table using the PerformResultsExport function.: "The database search results can be exported as a data frame table using the PerformResultsExport function."
- [readme] a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results: "3) a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results."
