---
name: spectral-library-search
description: Use when you have an IDSL.IPA-aligned peak table with individual sample
  peak lists, a reference MS/MS spectral library (e.g. NIST 2020), and you need to
  quantify how often IDSL.UFA's MS1 isotope-ranked formula hits agree with MS/MS-confirmed
  compound formulas.
when_to_use_negative:
- Input is only an MS2/DDA fragmentation file without the MS1 individual and aligned
  peak tables from IDSL.IPA — IDSL.UFA requires MS1 chromatographic peaklists for
  isotopic-profile ranking
- No reference MS/MS library (e.g. NIST 2020) is available, so MS/MS-confirmed compound
  identities cannot be established to benchmark formula ranks
- You need structural annotation rather than formula-level concordance — IDSL.UFA
  does not use MS2 spectra; it annotates only molecular formulas via MS1 isotopic
  profiles
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Extracts EIC candidates, detects peaks, performs recursive mass correction,
    RT correction and peak annotation to produce individual and aligned peaklists
    from raw mzML/netCDF for downstream formula ranking
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Annotates the IPA peaklists with ranked molecular formulas by matching theoretical
    vs observed isotopic profiles (MS1 only) and aggregates them on the aligned peak
    table
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.ExposomeDB
  role: Supplies the IPDB chemical space (209,592 and 129,122 ion formulas in positive
    and negative modes) used to generate the aligned annotated peak table for ST002044
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:72865d86bd6d4e5d59f5fef742809dbe08332577aa6c4406e37c583f76683f42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.ExposomeDB
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-library-search

## Summary

Reproduce MS/MS-supported annotation concordance on the ST002044 study by matching aligned peak table MS/MS spectra against the NIST 2020 library, then computing the median-rank distribution of IDSL.UFA molecular formula hits across individual samples to quantify how often the MS/MS-confirmed formula ranks at or near the top (78.75% of 73 confirmed hits had median rank ≤ 2).

## When to use

Use when you have an IDSL.IPA-aligned peak table with individual sample peak lists, a reference MS/MS spectral library (e.g. NIST 2020), and you need to quantify how often IDSL.UFA's MS1 isotope-ranked formula hits agree with MS/MS-confirmed compound formulas.

## When NOT to use

- Input is only an MS2/DDA fragmentation file without the MS1 individual and aligned peak tables from IDSL.IPA — IDSL.UFA requires MS1 chromatographic peaklists for isotopic-profile ranking
- No reference MS/MS library (e.g. NIST 2020) is available, so MS/MS-confirmed compound identities cannot be established to benchmark formula ranks
- You need structural annotation rather than formula-level concordance — IDSL.UFA does not use MS2 spectra; it annotates only molecular formulas via MS1 isotopic profiles

## Inputs

- Raw LC/HRMS data (mzXML, mzML, netCDF) for study ST002044 (positive and/or negative ESI mode)
- IDSL.IPA individual `peaklists` directory and aligned `peak_alignment` directory
- IDSL.UFA IPDB (.Rdata) isotopic-profile database, e.g. RefMetDB or IDSL.ExposomeDB
- NIST 2020 MS/MS spectral library for tandem-MS compound confirmation
- UFA parameter spreadsheet (UFA_parameters.xlsx) with PARAM0004–PARAM0014 configured

## Outputs

- median_rank_distribution.csv: count and percentage table of MS/MS-confirmed hits binned by median rank
- Per-peak annotated aligned peak table with molecular formula hits and their median ranks
- Concordance statistic: fraction of MS/MS-confirmed formula hits with median rank ≤ 2 (expected ≈78.75% on ST002044, 73 confirmed hits)

## How to apply

Run the IDSL.IPA workflow on ST002044 raw mzML/netCDF files to generate individual `peaklists` and the aligned `peak_alignment` directory, then run IDSL.UFA with an IPDB (e.g. RefMetDB, per PARAM0004) and PARAM0005/PARAM0006 = YES so candidate formulas are ranked in both individual and aligned tables. Match each aligned peak's MS/MS spectrum against the NIST 2020 library to identify the compound whose molecular formula is MS/MS-confirmed. For each confirmed hit, retrieve the formula's median rank across the aligned peak table generated using the IDSL.ExposomeDB IPDB (209,592 and 129,122 ion formulas in positive and negative modes), bin confirmed hits by median rank, and compute the fraction with median rank ≤ 2 — the article reports 78.75% (57/73). This validates that MS1 isotopic-profile ranking alone (no MS2 needed) recovers formulas corroborated by tandem mass spectrometry, addressing MSI-confidence annotation needs in exposomics where database-dependent tools miss poorly represented compounds.

## Related tools

- **IDSL.IPA** (Extracts EIC candidates, detects peaks, performs recursive mass correction, RT correction and peak annotation to produce individual and aligned peaklists from raw mzML/netCDF for downstream formula ranking) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Annotates the IPA peaklists with ranked molecular formulas by matching theoretical vs observed isotopic profiles (MS1 only) and aggregates them on the aligned peak table) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.ExposomeDB** (Supplies the IPDB chemical space (209,592 and 129,122 ion formulas in positive and negative modes) used to generate the aligned annotated peak table for ST002044)

## Examples

```
Rscript -e 'library(IDSL.UFA); UFA_workflow("UFA_parameters.xlsx")'
```

## Evaluation signals

- The aligned annotated peak table must summarize formula annotations using detection frequency and median ranks in multiple samples
- On ST002044, exactly 73 MS/MS-confirmed hits should be identified and the fraction with median rank ≤ 2 should be ~78.75% (57/73)
- Median-rank binning across individual samples should use the aligned peak table generated with the IDSL.ExposomeDB IPDB; re-running with a different IPDB (e.g. RefMetDB) changes the chemical space and hence the concordance rate
- IDSL.UFA must run with PARAM0005 and PARAM0006 set to YES so ranked formula hits are produced for both individual and aligned peak lists
- Cross-check concordance against the article's broader benchmark range: IDSL.UFA assigned 90.58–100% of true positive annotations within the top five hits

## Limitations

- IDSL.UFA annotates formulas using only MS1; it does not consume MS2 spectra, so MS/MS confirmation is used purely as an external benchmark rather than an internal input
- Concordance rates are IPDB-dependent: the IDSL.ExposomeDB IPDB chemical space (209,592 / 129,122 ion formulas) drives the ranking; smaller or biased formula sources may shift the median-rank distribution
- The article notes 'database dependent' approaches miss exposure compounds poorly represented in biochemical databases — formula-source choice directly affects which confirmed hits are reachable and how they rank
- Version of IDSL.UFA/IPA is not stated in the article text, and reproducibility details (exact parameters, NIST library build) are not fully specified

## Evidence

- [readme] The IDSL.UFA pipeline only requires MS1 for molecular formula annotation.: [evidence span withheld — anchor status: external_doc]
- [readme] UFA_workflow("Address of the UFA parameter spreadsheet"): [evidence span withheld — anchor status: external_doc]
