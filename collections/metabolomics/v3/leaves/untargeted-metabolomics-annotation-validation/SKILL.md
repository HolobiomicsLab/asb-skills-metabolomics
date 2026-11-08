---
name: untargeted-metabolomics-annotation-validation
description: Use when you have IDSL.IPA-generated individual and aligned peak tables
  (mzXML/mzML/netCDF inputs) and need to validate molecular formula assignments against
  MS/MS-confirmed identities or report median-rank statistics.
when_to_use_negative:
- Input is already a feature table without underlying raw MS1 data — IDSL.UFA requires
  raw mzXML/mzML/netCDF to match theoretical isotopic profiles; it is not a post-hoc
  annotation add-on for preprocessed tables.
- The goal is structure elucidation requiring MS2 fragmentation data only — IDSL.UFA
  performs formula annotation using MS1 isotopic profile matching; MS/MS-dependent
  tools (e.g. IDSL.CSA, SIRIUS) are the wrong fit for MS1-only formula assignment.
- Data are MS2-only or DIA/SWATH without MS1 isotopic envelopes — the isotopic profile
  matching approach assumes MS1-level isotope patterns for carbon-containing compounds.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: Generates individual peaklists and aligned peak tables (chromatographic m/z-RT
    information) from raw LC/HRMS data as input to formula annotation.
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Matches theoretical isotopic profiles (IPDB) against experimental MS1 data
    to rank molecular formula candidates and produce aligned annotated peak tables.
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.UFAx
  role: 'Exhaustive chemical enumeration fallback for formula annotation when a prioritized
    chemical space is unavailable (15 elements: C, B, Br, Cl, K, S, Si, N, H, As,
    F, I, Na, O, P).'
  repo: https://github.com/idslme/IDSL.UFAx
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:c695c33c23052ed23d53090389b17c8291f7cd9a91a337964426302cacf9c7fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9318181818181818
n_claims: 44
---

# untargeted-metabolomics-annotation-validation

## Summary

Validates molecular formula annotations on untargeted LC/HRMS peak tables by matching theoretical against experimental isotopic profiles (MS1) via IDSL.UFA, and independently cross-checks them against MS/MS library hits (e.g. NIST 2020) to quantify how often MS/MS-confirmed compounds receive a high-ranked formula hit.

## When to use

Use when you have IDSL.IPA-generated individual and aligned peak tables (mzXML/mzML/netCDF inputs) and need to validate molecular formula assignments against MS/MS-confirmed identities or report median-rank statistics.

## When NOT to use

- Input is already a feature table without underlying raw MS1 data — IDSL.UFA requires raw mzXML/mzML/netCDF to match theoretical isotopic profiles; it is not a post-hoc annotation add-on for preprocessed tables.
- The goal is structure elucidation requiring MS2 fragmentation data only — IDSL.UFA performs formula annotation using MS1 isotopic profile matching; MS/MS-dependent tools (e.g. IDSL.CSA, SIRIUS) are the wrong fit for MS1-only formula assignment.
- Data are MS2-only or DIA/SWATH without MS1 isotopic envelopes — the isotopic profile matching approach assumes MS1-level isotope patterns for carbon-containing compounds.

## Inputs

- Raw untargeted LC/HRMS data files (mzXML, mzML, netCDF) at MS1 level
- IDSL.IPA-generated `peaklists` and `peak_alignment` directories
- IPDB (.Rdata) theoretical isotopic profile database, e.g. RefMetDB positive/negative mode
- UFA parameter spreadsheet (UFA_parameters.xlsx)
- MS/MS spectra and a reference MS/MS library (e.g. NIST 2020 MS/MS library) for validation

## Outputs

- Aligned annotated peak table with per-peak molecular formula hits, ranks, and detection frequency
- Individual-sample annotated peak lists with formula median ranks
- Median-rank distribution table (e.g. fraction of MS/MS-confirmed hits with median rank ≤ 2), e.g. median_rank_distribution.csv
- Batch untargeted isotopic profile match figures

## How to apply

Run the IDSL.IPA workflow on raw LC/HRMS data (mzXML/mzML/netCDF) to generate the `peaklists` and `peak_alignment` directories, then download the UFA parameter spreadsheet and set PARAM0004 (IPDB address), PARAM0005/PARAM0006 = YES, PARAM0009 (MS1 data), PARAM0011 (peaklists), PARAM0012 (peak_alignment), and PARAM0014 (output) before calling `UFA_workflow`. IDSL.UFA computes theoretical isotopic profiles (IPDB) for molecular formulas and ranks candidate formulas per peak in both individual and aligned tables, summarizing annotations by detection frequency and median rank (e.g. top rank ≤ 30 and ≤ 30 hits). To validate, match MS/MS spectra of peaks against a reference library (e.g. NIST 2020 MS/MS) to confirm compound identities, then retrieve the confirmed compound's molecular formula among IDSL.UFA hits and compute the fraction of confirmed hits with median rank ≤ 2 (or ≤ 5) across individual samples in the aligned table; the article reports 54.31–85.51% true positives as top hit and 90.58–100% within the top five.

## Related tools

- **IDSL.IPA** (Generates individual peaklists and aligned peak tables (chromatographic m/z-RT information) from raw LC/HRMS data as input to formula annotation.) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Matches theoretical isotopic profiles (IPDB) against experimental MS1 data to rank molecular formula candidates and produce aligned annotated peak tables.) — https://github.com/idslme/IDSL.UFA
- **IDSL.UFAx** (Exhaustive chemical enumeration fallback for formula annotation when a prioritized chemical space is unavailable (15 elements: C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P).) — https://github.com/idslme/IDSL.UFAx

## Examples

```
library(IDSL.UFA); UFA_workflow('UFA_parameters.xlsx')
```

## Evaluation signals

- Fraction of MS/MS-confirmed molecular formula hits with median rank ≤ 2 (or ≤ 5) in the aligned annotated peak table — the article benchmarks 54.31–85.51% top-hit and 90.58–100% top-five true positives.
- Detection frequency and median rank summary statistics for each formula in the aligned table; check that top rank ≤ 30 and number of hits ≤ 30 filtering yields a coherent formula count (e.g. 199,837 unique formulas in one study).
- Compare IDSL.UFA formula assignments for standards against the experimental integrated isotopic profile spectra deposited at Zenodo (https://zenodo.org/record/5803968) to confirm theoretical-experimental match quality.
- Verify the output aligned table contains columns for formula, rank, and detection frequency and that no peak lacks a formula annotation when a valid IPDB was supplied.
- Cross-check that MS/MS-confirmed hits identified via the reference library appear among the IDSL.UFA candidate formulas for the corresponding m/z-RT peak.

## Limitations

- Database-dependent formula/annotation approaches are prone to missing exposure compounds poorly represented in biochemical databases and compounds lacking transformation products in DBs.
- The method assumes carbon-containing compounds with predictable natural isotopic distributions; MS1 isotopic envelope quality limits annotation confidence.
- The article does not state specific IDSL.UFA/IPA version numbers, so exact parameter reproducibility may vary across releases.
- Detection frequency and intensity thresholds used in some existing pipelines can filter out low-abundance exposome compounds, which IDSL.UFA's MS1-only approach partially mitigates but does not eliminate.

## Evidence

- [readme] The IDSL.UFA pipeline only requires MS1 for molecular formula annotation.: [evidence span withheld — anchor status: external_doc]
- [readme] To annotate your mass spectrometry data (mzXML, mzML, netCDF), mass spectrometry data should be processed using the IDSL.IPA workflow: [evidence span withheld — anchor status: external_doc]
- [methods] download the [IPA parameter spreadsheet] and select the parameters accordingly: [evidence span withheld — anchor status: not_found]
