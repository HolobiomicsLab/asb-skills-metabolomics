---
name: ms2-filtered-functional-interpretation
description: Use when raw LC-MS/MS data (DDA or SWATH-DIA mzML) have been processed
  to MS1 feature tables plus MS2 compound identifications, and the goal is to test
  whether MS2-informed functional analysis yields more significant pathways than MS1-only
  enrichment.
when_to_use_negative:
- Input is already a list of accurately identified, curated metabolites — traditional
  enrichment on named compounds does not need the empirical-compound MS2 filtering
  step
- Only MS1 data are available with no associated MS2 spectra, so no compound identifications
  can be used to filter candidates
- The goal is raw spectra processing or compound identification itself rather than
  downstream functional interpretation
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: R package providing Convert2Mummichog for the paired MS1-only vs MS2-filtered
    pathway enrichment runs, plus the upstream MS2 identification functions feeding
    the filter
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: PerformDBSearchingBatch
  role: SQLite-based MS2 database searching that produces the compound identifications
    used to filter unfeasible empirical-compound candidates
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: PerformResultsExport
  role: Exports the database search results as a data frame table for use in the mummichog
    step
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: PerformSpectrumConsenus
  role: Builds consensus MS2 spectra across replicates (50% fragment frequency threshold)
    prior to identification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:94a485b133ad7e0fcf3680d2474a4aa1adc66aa057c6e31bc0091d303da415aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - PerformDBSearchingBatch
  - PerformResultsExport
  - PerformSpectrumConsenus
  - R
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms2-filtered-functional-interpretation

## Summary

Run mummichog-style pathway/functional enrichment twice — once on MS1 features alone and once with MS2-based compound identifications used to remove unfeasible empirical-compound candidates before the permutation test — to quantify how spectral annotation improves biological insight.

## When to use

Use when raw LC-MS/MS data (DDA or SWATH-DIA mzML) have been processed to MS1 feature tables plus MS2 compound identifications, and the goal is to test whether MS2-informed functional analysis yields more significant pathways than MS1-only enrichment.

## When NOT to use

- Input is already a list of accurately identified, curated metabolites — traditional enrichment on named compounds does not need the empirical-compound MS2 filtering step
- Only MS1 data are available with no associated MS2 spectra, so no compound identifications can be used to filter candidates
- The goal is raw spectra processing or compound identification itself rather than downstream functional interpretation

## Inputs

- Complete MS1 feature table (m/z, retention time, intensity) from MetaboAnalystR 4.0 auto-optimized peak picking/alignment/gap filling
- MS2-based compound identification results exported as a data frame via PerformResultsExport
- Consensus MS2 spectra built across replicates with a 50% fragment frequency threshold
- Raw centroided mzML files (e.g., Metabolomics Workbench ST002796 and ST002798 clinical COVID-19 datasets)

## Outputs

- Pathway enrichment results table from the MS1-only (unfiltered) mummichog run
- Pathway enrichment results table from the MS2-filtered mummichog run
- Side-by-side comparison table (e.g., .csv) reproducing the Fig. 5c-e style unfiltered-vs-filtered comparison

## How to apply

First process LC-MS1 spectra with MetaboAnalystR 4.0's auto-optimized workflow (peak picking, alignment, gap filling, annotation) to obtain a complete MS1 feature table, and identify compounds by deconvolving MS2 spectra (PerformDDADeconvolution for DDA, PerformDIADeconvolution for SWATH-DIA), building replicate consensus spectra with PerformSpectrumConsenus at a 50% fragment frequency threshold, and searching curated SQLite reference libraries via PerformDBSearchingBatch. Then call Convert2Mummichog twice: once passing only the MS1 feature list (no MS2 filtering), and once supplying the MS2-based compound identifications so that empirical-compound candidates inconsistent with spectral evidence are excluded before the permutation test. Finally, compile the pathway enrichment results from both runs side by side; the filtered run should recover pathways masked in the unfiltered analysis (in the COVID-19 SWATH-DIA case study, four additional significant pathways: phosphatidylinositol phosphate metabolism, vitamin D, vitamin C, and arachidonic cascade). The rationale is that traditional functional enrichment relies on accurate compound identification, and MS2 annotation debiases the empirical-compound space that mummichog permutes over.

## Related tools

- **MetaboAnalystR 4.0** (R package providing Convert2Mummichog for the paired MS1-only vs MS2-filtered pathway enrichment runs, plus the upstream MS2 identification functions feeding the filter) — https://github.com/xia-lab/MetaboAnalystR
- **PerformDBSearchingBatch** (SQLite-based MS2 database searching that produces the compound identifications used to filter unfeasible empirical-compound candidates) — https://github.com/xia-lab/MetaboAnalystR
- **PerformResultsExport** (Exports the database search results as a data frame table for use in the mummichog step) — https://github.com/xia-lab/MetaboAnalystR
- **PerformSpectrumConsenus** (Builds consensus MS2 spectra across replicates (50% fragment frequency threshold) prior to identification)

## Examples

```
mSet <- Convert2Mummichog(mSet, ms2.ident = res_df, filter.mode = "ms2")  # run once without ms2.ident (MS1-only) and once with MS2 filtering
```

## Evaluation signals

- The comparison output exists as a .csv and clearly distinguishes the unfiltered vs MS2-filtered runs
- For the SWATH-DIA clinical COVID-19 dataset, the MS2-filtered run reports four additional significant pathways (phosphatidylinositol phosphate metabolism, vitamin D, vitamin C, and arachidonic cascade) relative to MS1-only
- The two mummichog runs used identical MS1 feature lists, differing only in whether MS2-based compound identifications filtered the empirical-compound candidates before the permutation test
- MS2 identifications derive from consensus spectra meeting the 50% fragment frequency threshold across replicates
- The filtered run does not lose pathways that were significant in the MS1-only run (filtering should increase or maintain, not reduce, biological coverage)

## Limitations

- MS2-based compound identification depends heavily on the availability of reference spectra databases, which are currently incomplete and compiled from varying instruments and experimental conditions, resulting in potentially varying qualities
- Gains depend on acquisition type — in the case study, the four additional pathways were demonstrated on the SWATH-DIA dataset, not the DDA dataset
- Stable isotope-labeling analysis is not yet available in the framework

## Evidence

- [abstract] more accurate functional interpretation through integrated spectral annotation: "more accurate functional interpretation through integrated spectral annotation"
- [intro] Traditional functional enrichment analysis relies on accurate compound identification.: "Traditional functional enrichment analysis relies on accurate compound identification."
- [methods] If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed.: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [readme] For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications.: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
