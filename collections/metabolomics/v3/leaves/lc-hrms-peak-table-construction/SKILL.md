---
name: lc-hrms-peak-table-construction
description: Use when you have IDSL.IPA per-file formula annotation peaklists plus
  a peak_alignment directory for an LC/HRMS study and need a per-peak summary annotation
  (detection frequency + median rank) exported as a csv.
when_to_use_negative:
- Input is already the aligned annotated peak table with detection frequency and median
  rank columns computed - no further aggregation step is needed
- Structural annotation requiring MS2/fragmentation spectra is the goal (composite
  spectra, DDA, or DIA analyses should route to IDSL.CSA-type workflows)
- No per-file formula annotation peaklists exist because raw data has not been peak-picked
  (run the IDSL.IPA peak-detection workflow first)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: 'Peak-picking and peak-alignment stage: produces the individual peaklists
    and peak_alignment directory that the aggregation step consumes'
  repo: github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Performs the alignment-table aggregation itself - annotates peaklists with
    molecular formulas via isotopic profile matching and summarizes formulas with
    detection frequency and median ranks in the aligned annotated peak table
  repo: github.com/idslme/IDSL.UFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:46b3c13803ff8a5137ca602c8cb05c28d11aa5f1efcb720015a194e132d77daf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-hrms-peak-table-construction

## Summary

Aggregate per-sample molecular formula annotations from IDSL.IPA-generated peaklists into a single aligned annotated peak table, ranking each aligned peak's formula candidates by detection frequency and median rank across samples.

## When to use

Use when you have IDSL.IPA per-file formula annotation peaklists plus a peak_alignment directory for an LC/HRMS study and need a per-peak summary annotation (detection frequency + median rank) exported as a csv.

## When NOT to use

- Input is already the aligned annotated peak table with detection frequency and median rank columns computed - no further aggregation step is needed
- Structural annotation requiring MS2/fragmentation spectra is the goal (composite spectra, DDA, or DIA analyses should route to IDSL.CSA-type workflows)
- No per-file formula annotation peaklists exist because raw data has not been peak-picked (run the IDSL.IPA peak-detection workflow first)

## Inputs

- Per-file formula annotation peaklists from IDSL.IPA (peaklists directory)
- Peak alignment table from IDSL.IPA (peak_alignment directory)
- IDSL.UFA parameter spreadsheet (30 parameters across 4 sections) with PARAM0006 enabled
- IPDB (.Rdata) theoretical isotopic profile database (e.g. RefMetDB positive or negative mode)
- MS1-level HRMS data files (mzXML, mzML, netCDF)

## Outputs

- Aligned annotated peak table (csv) with detection frequency and median rank per formula per aligned peak, sorted by the new frequency/median-rank order
- Per-peak formula candidate lists retaining top 5-20 ranked hits

## How to apply

Run the IDSL.IPA workflow over the MS1-level HRMS data files (mzXML, mzML, netCDF) to obtain individual peaklists and the peak_alignment table; configure the IDSL.UFA parameter spreadsheet with PARAM0005 and PARAM0006 = YES so aggregation on the aligned peak table is performed, and point PARAM0011/PARAM0012 at the peaklists and peak_alignment directories. For each aligned peak, count in how many samples each candidate formula was annotated (detection frequency) and compute the median rank of each formula across those samples. Re-sort formula candidates per peak using these two summary statistics and export the aligned annotated peak table as a csv retaining the top 5-20 formula hits per peak. When reporting unique formulas from the aligned table, apply cutoffs such as top rank ≤ 30 and number of hits ≤ 30, and flag peaks with at least one formula of median rank ≤ 5.

## Related tools

- **IDSL.IPA** (Peak-picking and peak-alignment stage: produces the individual peaklists and peak_alignment directory that the aggregation step consumes) — github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Performs the alignment-table aggregation itself - annotates peaklists with molecular formulas via isotopic profile matching and summarizes formulas with detection frequency and median ranks in the aligned annotated peak table) — github.com/idslme/IDSL.UFA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Rows of the exported csv correspond 1:1 with aligned peaks in the IDSL.IPA peak_alignment table
- Each retained formula row carries two summary statistics: a detection frequency across samples and a median rank; rows absent either column indicate the aggregation step (PARAM0006) did not run
- Reported peaks pass the median rank ≤ 5 threshold; unique formulas reported use the top rank ≤ 30 and number of hits ≤ 30 filters (paper benchmark: 199,837 unique formulas)
- Top 5-20 formula hits per peak are retained; more than ~20 hits per row indicates retention cutoffs were not applied
- Sort order of formula candidates per peak changes after aggregation relative to the per-file ranks, confirming the frequency/median-rank re-ranking took effect

## Limitations

- Aggregation on the aligned peak table is presented only in IDSL.UFA; other peak-annotation pipelines do not implement this statistic, so results are not cross-tool reproducible
- Annotation relies on MS1 only and the chosen IPDB formula source; database-dependent formula sources risk missing exposure compounds poorly represented in biochemical databases
- Paper does not state IDSL.UFA/IDSL.IPA versions or full parameter reproducibility details for this section

## Evidence

- [readme] Aggregating annotated molecular formulas on the aligned peak table is unique to IDSL.UFA: [evidence span withheld — anchor status: external_doc]
- [readme] Peaklists from IDSL.IPA are annotated by UFA using isotopic profile matching with MS1 only: [evidence span withheld — anchor status: not_found]
