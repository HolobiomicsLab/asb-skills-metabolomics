---
name: molecular-formula-isotopic-profile-matching
description: Use when you have MS1-level LC/HRMS data (mzXML, mzML, netCDF) with chromatographic
  peaks and need high-confidence molecular formula annotations via isotopic profile
  matching rather than database-dependent MS2 search.
when_to_use_negative:
- When the goal is exhaustive de novo elemental composition enumeration across vast
  chemical spaces (e.g., 15 elements C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O,
  P) — use IDSL.UFAx instead of IPDB-based matching
- When peaks already carry trusted MSI 1–3 formula annotations from spectral library
  search and isotopic re-annotation adds no value
- When only MS2 fragmentation spectra are available without MS1 isotopic profiles
  — IDSL.UFA's pipeline only requires MS1 but cannot operate without it
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: 'Core engine: computes theoretical isotopic profiles for molecular formulas
    from an IPDB, matches them against experimental MS1 data, ranks candidate hits,
    and builds the aligned annotated peak table'
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.IPA
  role: 'Upstream peak picking: extracts EIC candidates, detects peaks, performs recursive
    mass correction, RT correction across batches, and generates the peaklists/peak_alignment
    directories consumed by UFA'
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFAx
  role: Complementary exhaustive chemical enumeration-based annotator for a subset
    of peaks when a prioritized formula source is insufficient
  repo: https://github.com/idslme/IDSL.UFAx
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:57f669e3388444b9797f5057dee4a0bba558acca38d5685607df9c3f5c524483
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.UFA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-formula-isotopic-profile-matching

## Summary

Assigns molecular formulas to peaks in untargeted LC/HRMS metabolomics/exposomics datasets by matching theoretical isotopic profiles against experimental MS1 isotopic profiles using the IDSL.UFA R package, with ranked candidate hits and sample-level aggregation.

## When to use

Use when you have MS1-level LC/HRMS data (mzXML, mzML, netCDF) with chromatographic peaks and need high-confidence molecular formula annotations via isotopic profile matching rather than database-dependent MS2 search.

## When NOT to use

- When the goal is exhaustive de novo elemental composition enumeration across vast chemical spaces (e.g., 15 elements C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) — use IDSL.UFAx instead of IPDB-based matching
- When peaks already carry trusted MSI 1–3 formula annotations from spectral library search and isotopic re-annotation adds no value
- When only MS2 fragmentation spectra are available without MS1 isotopic profiles — IDSL.UFA's pipeline only requires MS1 but cannot operate without it

## Inputs

- MS1-level untargeted LC/HRMS raw data files (mzXML, mzML, netCDF)
- IDSL.IPA 'peaklists' directory (per-file chromatographic peak m/z-RT information)
- IDSL.IPA 'peak_alignment' directory (aligned peak table)
- IPDB .Rdata theoretical isotopic profile database for a prioritized chemical space (e.g., RefMetDB positive/negative mode)
- UFA parameter spreadsheet (UFA_parameters.xlsx) with filled PARAM0004-PARAM0014 settings

## Outputs

- Ranked molecular formula annotations per peak (individual annotated peak lists)
- Aligned annotated peak table summarizing formulas by detection frequency and median ranks
- Batch untargeted isotopic profile match figures
- Per-compound validation metrics (NEME in mDa, pairwise cosine similarity PCS %)

## How to apply

First process raw MS1 HRMS data with IDSL.IPA to generate 'peaklists' and 'peak_alignment' directories containing chromatographic m/z-RT information. Obtain or build an IPDB (.Rdata) of theoretical isotopic profiles computed from natural isotopic distributions (IUPAC atomic mass tables) for your prioritized formula source (e.g., RefMetDB). Download the UFA parameter spreadsheet (~30 parameters in 4 sections) and set PARAM0004 (IPDB address), PARAM0005 and PARAM0006 to YES (to enable aligned-peak-table aggregation), PARAM0009 (MS1 data location), PARAM0011 (peaklists directory), PARAM0012 (peak_alignment directory), PARAM0014 (output location), and optionally PARAM0008 (threads). Run UFA_workflow on the spreadsheet; UFA ranks candidate formulas per peak by matching the theoretical isotopic profile to the observed MS1 profile, and summarizes annotations across samples using detection frequency and median ranks in an aligned annotated peak table. Validate on authentic reference standards (MSV000088661) by computing normalized Euclidean mass error (NEME) and profile cosine similarity (PCS); the paper reports mean NEME of 0.70 mDa and mean PCS of 99.968% across 367 standards.

## Related tools

- **IDSL.UFA** (Core engine: computes theoretical isotopic profiles for molecular formulas from an IPDB, matches them against experimental MS1 data, ranks candidate hits, and builds the aligned annotated peak table) — https://github.com/idslme/IDSL.UFA
- **IDSL.IPA** (Upstream peak picking: extracts EIC candidates, detects peaks, performs recursive mass correction, RT correction across batches, and generates the peaklists/peak_alignment directories consumed by UFA) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFAx** (Complementary exhaustive chemical enumeration-based annotator for a subset of peaks when a prioritized formula source is insufficient) — https://github.com/idslme/IDSL.UFAx

## Examples

```
UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- On 367 authentic reference standard compounds (MSV000088661, Zenodo 5803968), aggregate mean NEME ≈ 0.70 mDa between assigned formulas' monoisotopic masses
- Aggregate mean pairwise cosine similarity (PCS) between theoretical and experimental isotopic profiles ≈ 99.968%
- True positive formulas ranked top hit 54.31–85.51% of the time and within the top five hits 90.58–100% of the time
- Unique molecular formulas in the aligned table reported with top rank ≤ 30 and number of hits ≤ 30; annotated aligned peaks flagged when at least one formula has median rank ≤ 5
- Aligned annotated peak table contains non-empty detection frequency and median rank summary columns (enabled via PARAM0005 and PARAM0006 = YES)

## Limitations

- Database-dependent formula sources can miss exposure compounds poorly represented in biochemical databases and compounds lacking transformation products; IPDB choice bounds the searchable chemical space
- Annotations are formula-level only; structural identity requires complementary MS2-based annotation pipelines
- Article reproducibility gaps: no IDSL.UFA/IDSL.IPA version stated and some sections omit parameter and reference-database details

## Evidence

- [results] Across 367 authentic standard compounds, IDSL.UFA achieved an average NEME of 0.70 mDa and an average PCS of 99.968% between experimental and theoretical isotopic profiles.: [evidence span withheld — anchor status: not_found]
- [readme] The IDSL.UFA pipeline only requires MS1 for molecular formula annotation.: [evidence span withheld — anchor status: external_doc]
