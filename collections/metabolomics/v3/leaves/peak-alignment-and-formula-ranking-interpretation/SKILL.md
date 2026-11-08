---
name: peak-alignment-and-formula-ranking-interpretation
description: Use when you have IDSL.IPA peaklists and an aligned peak table processed
  by IDSL.UFA with a candidate IPDB, and need to quantify formula-annotation performance
  (top-hit and top-five-hit recovery) against known MSI 1-3 annotations.
when_to_use_negative:
- Your peaks already carry MS2/MSI level-1 structural identifications and you need
  structural rather than formula-level benchmarking.
- No IPDB chemical space is appropriate and you need exhaustive elemental enumeration
  instead of database-constrained isotopic matching (use IDSL.UFAx).
- The LC/HRMS data have not been processed by IDSL.IPA, so no peaklists or peak_alignment
  directory exist to annotate.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: 'Peak extraction and alignment: runs IPA_workflow/IPA_targeted on raw LC/HRMS
    data to produce peaklists and the aligned peak table that IDSL.UFA annotates.'
  repo: github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Assigns and ranks molecular formulas per peak by matching theoretical isotopic
    profiles (from the IPDB) against experimental MS1 isotopic profiles; requires
    only MS1 data.
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: R
  role: Runtime environment for the UFA_workflow/IPA_workflow calls and for computing
    recovery fractions and compiling the comparison table.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:d8f5cd803123d7ed6954d2acefc6ce7fdbd4ad39348d801fa3d06969f6137fd2
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-alignment-and-formula-ranking-interpretation

## Summary

Benchmarking and interpreting ranked molecular formula annotations on IDSL.IPA-generated aligned peak tables from IDSL.UFA, by recovering MSI 1-3 annotated peaks and computing top-hit and top-5-hit formula assignment rates against a chosen IPDB chemical space.

## When to use

Use when you have IDSL.IPA peaklists and an aligned peak table processed by IDSL.UFA with a candidate IPDB, and need to quantify formula-annotation performance (top-hit and top-five-hit recovery) against known MSI 1-3 annotations.

## When NOT to use

- Your peaks already carry MS2/MSI level-1 structural identifications and you need structural rather than formula-level benchmarking.
- No IPDB chemical space is appropriate and you need exhaustive elemental enumeration instead of database-constrained isotopic matching (use IDSL.UFAx).
- The LC/HRMS data have not been processed by IDSL.IPA, so no peaklists or peak_alignment directory exist to annotate.

## Inputs

- Raw MS1-level LC/HRMS data files (mzXML, mzML, netCDF)
- IDSL.IPA outputs: peaklists directory and peak_alignment directory (aligned peak table)
- IPA parameter spreadsheet and UFA parameter spreadsheet (UFA_parameters.xlsx, 30 parameters in 4 sections)
- IPDB (.Rdata) theoretical isotopic profile database, e.g. IDSL.ExposomeDB with 209,592 and 129,122 ion formulas in positive and negative modes
- Reference MSI 1-3 annotated peak table (e.g. annotations from Zenodo deposit 5838709)

## Outputs

- Aligned annotated peak table with ranked molecular formula hits per peak (median ranks and detection frequencies across samples)
- Per-study top-hit and top-5-hit formula recovery fractions over MSI 1-3 true-positive annotations
- Comparison table of assignment rates across studies

## How to apply

First process raw mzXML/mzML/netCDF MS1 data with the IDSL.IPA IPA_workflow or IPA_targeted functions to generate per-file peaklists and the peak_alignment directory. Then configure the UFA parameter spreadsheet (PARAM0004 pointing to the IPDB .Rdata such as IDSL.ExposomeDB with 209,592 positive-mode and 129,122 negative-mode ion formulas, PARAM0009 to the raw MS1 data, PARAM0011/PARAM0012 to the peaklists and peak_alignment directories, PARAM0005 and PARAM0006 = YES) and run UFA_workflow, which matches theoretical isotopic profiles against experimental LC/HRMS data in each file. Restrict reported formulas on the aligned annotated peak table to top rank ≤ 30 and number of hits ≤ 30, and report annotations with at least one formula having a median rank ≤ 5. Match assigned formulas against the MSI 1-3 true-positive annotations (e.g. from the MTBLS2542 Zenodo deposit 5838709) and compute, per study, the fraction of true positives recovered as the top-ranked hit and within the top five ranked hits. Compile the per-study rates into a comparison table; for the four public studies against IDSL.ExposomeDB these windows are 54.31-85.51% (top hit) and 90.58-100% (top five hits).

## Related tools

- **IDSL.IPA** (Peak extraction and alignment: runs IPA_workflow/IPA_targeted on raw LC/HRMS data to produce peaklists and the aligned peak table that IDSL.UFA annotates.) — github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Assigns and ranks molecular formulas per peak by matching theoretical isotopic profiles (from the IPDB) against experimental MS1 isotopic profiles; requires only MS1 data.) — https://cran.r-project.org/package=IDSL.UFA
- **R** (Runtime environment for the UFA_workflow/IPA_workflow calls and for computing recovery fractions and compiling the comparison table.)

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Per-study top-hit rates fall within the 54.31-85.51% window and top-5-hit rates within the 90.58-100% window when using the IDSL.ExposomeDB IPDB (ST001154: 61.85% / 96.90%; ST001683: 54.31% / 90.58%; MTBLS1684: 70.58% / 100%; MTBLS2542: 85.51% / 99.29%).
- Reported formulas on the aligned table satisfy top rank ≤ 30 and number of hits ≤ 30.
- Annotations used for rate computation have a median rank ≤ 5 in the aligned annotated peak table.
- Reference true positives are MSI 1-3 level annotations traceable to Zenodo deposit 5838709.
- IDSL.UFA ran with PARAM0004 set to the IPDB .Rdata and PARAM0009 pointing at MS1-level raw data.

## Limitations

- IDSL.UFA annotation uses only MS1 isotopic-profile matching; structural confirmation additionally requires tandem MS support.
- Recovery rates are database-dependent: formula assignment searches only the IPDB chemical space, and the reported 54.31-85.51% / 90.58-100% windows were established for four specific public studies against IDSL.ExposomeDB.
- The top-hit rate never reaches 100% even on known MSI 1-3 annotations, so ranked-hit interpretation (not top-rank-only matching) is required for sensitive reporting.

## Evidence

- [readme] The parameter spreadsheet is the driver of the IDSL.UFA workflow.: [evidence span withheld — anchor status: external_doc]
