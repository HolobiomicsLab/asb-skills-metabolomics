---
name: molecular-formula-annotation-lc-hrms
description: Use when you have IDSL.IPA peaklists (mzXML/mzML/netCDF-derived m/z-RT
  peaks) from untargeted LC/HRMS data and need MS1-level molecular formula assignments
  ranked against a chemical-space database such as IDSL.ExposomeDB.
when_to_use_negative:
- You need de novo elemental composition without a formula database — use IDSL.UFAx
  exhaustive enumeration (C, H, N, O, P, S, halogens, etc.) instead of IPDB-dependent
  IDSL.UFA
- Peak chromatographic information has not yet been extracted — IDSL.UFA requires
  IDSL.IPA peaklists and aligned peak tables as input, not raw spectra alone
- Structural identification requiring MS2 spectra is the goal — IDSL.UFA annotates
  molecular formulas from MS1 isotopic profiles only and does not assign structures
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.UFA
  role: 'Core R package: computes theoretical isotopic profiles for molecular formulas
    from an IPDB, matches them against experimental MS1 isotopic profiles, and ranks
    formula candidates for individual and aligned peak lists'
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: 'Upstream peak extraction: generates peaklists (m/z-RT) and the aligned peak
    table from raw mzXML/mzML/netCDF LC/HRMS data that IDSL.UFA annotates; run via
    IPA_workflow or IPA_targeted'
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: 'Execution environment: install.packages(''IDSL.UFA'') and call UFA_workflow
    on the parameter spreadsheet'
  repo: https://cran.r-project.org/package=IDSL.UFA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:f7e37e3e721f74da87bfeb57ee974d326d101217d87b3f4f2bf035f9fe754336
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# molecular-formula-annotation-lc-hrms

## Summary

Assign high-confidence molecular formulas to peaks in untargeted LC/HRMS metabolomics/exposomics datasets by matching experimental MS1 isotopic profiles against theoretical isotopic profiles computed by IDSL.UFA from a chosen IPDB formula source.

## When to use

Use when you have IDSL.IPA peaklists (mzXML/mzML/netCDF-derived m/z-RT peaks) from untargeted LC/HRMS data and need MS1-level molecular formula assignments ranked against a chemical-space database such as IDSL.ExposomeDB.

## When NOT to use

- You need de novo elemental composition without a formula database — use IDSL.UFAx exhaustive enumeration (C, H, N, O, P, S, halogens, etc.) instead of IPDB-dependent IDSL.UFA
- Peak chromatographic information has not yet been extracted — IDSL.UFA requires IDSL.IPA peaklists and aligned peak tables as input, not raw spectra alone
- Structural identification requiring MS2 spectra is the goal — IDSL.UFA annotates molecular formulas from MS1 isotopic profiles only and does not assign structures

## Inputs

- Raw untargeted LC/HRMS data files (mzXML, mzML, netCDF), MS1 level
- IDSL.IPA-generated peaklists directory and peak_alignment directory (m/z-RT chromatographic peak information)
- UFA parameter spreadsheet (UFA_parameters.xlsx, 30 parameters in 4 sections)
- Pre-calculated IPDB (.Rdata) theoretical isotopic profile database for the chosen formula source (e.g. IDSL.ExposomeDB IPDB)

## Outputs

- Annotated individual peak tables with ranked candidate molecular formulas per peak
- Aligned annotated peak table aggregating formulas by detection frequency and median rank across samples
- Batch untargeted isotopic profile match figures
- Benchmark table of top-hit and top-5-hit formula assignment rates against MSI 1-3 true positives

## How to apply

First process raw MS1-level LC/HRMS data with IDSL.IPA (IPA_workflow, or IPA_targeted for large peak counts) to generate individual peaklists and the aligned peak table; download the UFA/IPA parameter spreadsheet and fill PARAM0004 (IPDB .Rdata address), PARAM0005/PARAM0006 = YES, PARAM0009 (MS1 data address), PARAM0011 (peaklists directory), PARAM0012 (peak_alignment directory), PARAM0014 (output location). Select an IPDB formula source appropriate to the chemical space of interest — e.g. the deposited IDSL.ExposomeDB IPDB with 209,592 positive-mode and 129,122 negative-mode ion formulas — then run UFA_workflow on the parameter spreadsheet; IDSL.UFA computes theoretical isotopic profiles using IUPAC atomic mass tables and matches them to observed MS1 isotopologue patterns, ranking candidate formulas per peak. Aggregate annotations into an aligned annotated peak table, summarizing each formula by detection frequency and median rank across samples; report unique formulas with top rank ≤ 30 and number of hits ≤ 30, and prioritize formulas with median rank ≤ 5. Validate by benchmarking against MSI 1-3 annotated true positives (e.g. from Zenodo deposit 5838709), expecting top-hit rates around 54.31-85.51% and top-five-hit rates around 90.58-100% per study.

## Related tools

- **IDSL.UFA** (Core R package: computes theoretical isotopic profiles for molecular formulas from an IPDB, matches them against experimental MS1 isotopic profiles, and ranks formula candidates for individual and aligned peak lists) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream peak extraction: generates peaklists (m/z-RT) and the aligned peak table from raw mzXML/mzML/netCDF LC/HRMS data that IDSL.UFA annotates; run via IPA_workflow or IPA_targeted) — https://github.com/idslme/IDSL.IPA
- **R** (Execution environment: install.packages('IDSL.UFA') and call UFA_workflow on the parameter spreadsheet) — https://cran.r-project.org/package=IDSL.UFA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Benchmarking against MSI 1-3 annotated peaks (e.g. studies ST001154, ST001683, MTBLS1684, MTBLS2542) should yield top-hit assignment rates in the 54.31-85.51% range
- The same benchmark should yield top-five-hit assignment rates in the 90.58-100% range
- Aligned-table formula aggregation should report unique molecular formulas filtered at top rank ≤ 30 and number of hits ≤ 30
- Reported/prioritized formulas should carry a median rank ≤ 5 in at least the samples where detected
- IPDB selection must match ionization mode: the IDSL.ExposomeDB IPDB contains 209,592 positive-mode and 129,122 negative-mode ion formulas

## Limitations

- Formula annotation depends on the IPDB chemical space chosen: 'database dependent' approaches risk missing exposure compounds poorly represented in biochemical databases, compounds lacking transformation products, and compounds filtered out by detection frequency/intensity thresholds
- Requires only MS1 data — assignments are formula-level and are not structural identifications, though they can complement MS2-based peak annotation pipelines
- Software versions of IDSL.UFA/IDSL.IPA and some reproducibility parameters are not stated in all contexts

## Evidence

- [readme] To annotate your mass spectrometry data (mzXML, mzML, netCDF), mass spectrometry data should be processed using the IDSL.IPA workflow to acquire chromatographic information of the peaks (m/z-RT): [evidence span withheld — anchor status: external_doc]
