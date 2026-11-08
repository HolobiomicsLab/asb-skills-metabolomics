---
name: cross-study-annotation-rate-benchmarking
description: Use when you have publicly available LC/HRMS studies with MSI 1–3 peak
  annotations plus a tool that produces ranked molecular-formula assignments per peak,
  and you need to quantify top-hit and top-5-hit recovery rates to compare formula-annotation
  performance across studies.
when_to_use_negative:
- The tool under test requires MS2 spectra or MS/MS-based formula assignment; IDSL.UFA
  operates on MS1-only data via isotopic profile matching.
- The goal is de novo formula enumeration without a reference chemical database; use
  IDSL.UFAx (exhaustive chemical enumeration over C, B, Br, Cl, K, S, Si, N, H, As,
  F, I, Na, O, P) instead.
- The studies lack public MSI 1–3 annotations to serve as ground-truth true positives,
  making top-hit/top-five recovery rates unmeasurable.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: IDSL.UFA
  role: Isotopic-profile matching tool that assigns ranked molecular formulas to LC/HRMS
    peaks; run per study and queried for top-hit / top-five recovery against MSI 1–3
    annotations
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Upstream peak-picking and alignment step that generates individual peaklists
    and the aligned peak table fed to IDSL.UFA for formula annotation
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Language environment in which IDSL.UFA and IDSL.IPA are run as R packages
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.ExposomeDB
  role: Source of the IPDB containing 209,592 and 129,122 ion formulas in positive
    and negative modes, used as the formula target space for annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:8fecd88b8b9a75939def7ddd7d3258ecc7f3abb853e2341adee4f9cccd7a3765
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.ExposomeDB
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-study-annotation-rate-benchmarking

## Summary

Benchmarking protocol that quantifies how many MSI 1–3 annotated LC/HRMS peaks from public metabolomics studies (e.g. ST001154, ST001683, MTBLS1684, MTBLS2542) receive a molecular formula as the top hit and within the top five hits from an isotopic-profile formula-annotation tool (IDSL.UFA) run against a fixed IPDB (IDSL.ExposomeDB).

## When to use

Use when you have publicly available LC/HRMS studies with MSI 1–3 peak annotations plus a tool that produces ranked molecular-formula assignments per peak, and you need to quantify top-hit and top-5-hit recovery rates to compare formula-annotation performance across studies.

## When NOT to use

- The tool under test requires MS2 spectra or MS/MS-based formula assignment; IDSL.UFA operates on MS1-only data via isotopic profile matching.
- The goal is de novo formula enumeration without a reference chemical database; use IDSL.UFAx (exhaustive chemical enumeration over C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) instead.
- The studies lack public MSI 1–3 annotations to serve as ground-truth true positives, making top-hit/top-five recovery rates unmeasurable.

## Inputs

- Raw LC/HRMS data files (mzXML, mzML, netCDF) from public metabolomics studies (e.g. ST001154, ST001683, MTBLS1684, MTBLS2542)
- MSI 1–3 annotated peak lists from the corresponding deposit (e.g. Zenodo 5838709 for MTBLS2542)
- IDSL.UFA parameter spreadsheet (UFA_parameters.xlsx)
- Isotopic profile database (IPDB, e.g. IDSL.ExposomeDB with 209,592 positive-mode and 129,122 negative-mode ion formulas)
- IDSL.IPA-generated peaklists and peak_alignment directories

## Outputs

- Per-study top-hit annotation rate (% of MSI 1–3 peaks with formula ranked #1)
- Per-study top-five-hit annotation rate (% of MSI 1–3 peaks with formula ranked within top 5)
- Aligned annotated peak table with molecular formula rankings (median rank ≤ 5)
- Cross-study comparison table summarizing annotation rates

## How to apply

1) Download the public LC/HRMS raw data and the deposited IDSL.UFA parameter spreadsheet plus the target IPDB (IDSL.ExposomeDB: 209,592 and 129,122 ion formulas in positive and negative modes respectively, from Zenodo accession 5838709). 2) Configure the UFA parameter spreadsheet (PARAM0004 = IPDB address, PARAM0009 = MS1 HRMS data location, PARAM0011 = IDSL.IPA peaklists directory, PARAM0012 = IDSL.IPA peak_alignment directory, PARAM0014 = output location) and run IDSL.IPA's IPA_workflow/IPA_targeted on each study to produce individual peaklists and aligned annotated peak tables. 3) Run IDSL.UFA with top rank ≤ 30 and number of hits ≤ 30, then report only annotations with median rank ≤ 5. 4) Cross-reference assigned formulas against the MSI 1–3 annotations deposited for MTBLS2542 (Zenodo 5838709) to identify true-positive peaks. 5) For each study, compute the fraction of true-positive annotations receiving a formula as the top hit and within the top five hits. 6) Compile a per-study comparison table and verify that top-hit rates fall in the 54.31–85.51% range and top-five rates fall in the 90.58–100% range as reported for IDSL.UFA.

## Related tools

- **IDSL.UFA** (Isotopic-profile matching tool that assigns ranked molecular formulas to LC/HRMS peaks; run per study and queried for top-hit / top-five recovery against MSI 1–3 annotations) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream peak-picking and alignment step that generates individual peaklists and the aligned peak table fed to IDSL.UFA for formula annotation) — https://github.com/idslme/IDSL.IPA
- **R** (Language environment in which IDSL.UFA and IDSL.IPA are run as R packages) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.ExposomeDB** (Source of the IPDB containing 209,592 and 129,122 ion formulas in positive and negative modes, used as the formula target space for annotation)

## Examples

```
library(IDSL.UFA); UFA_workflow("UFA_parameters.xlsx")  # after IDSL.IPA has produced peaklists and peak_alignment directories
```

## Evaluation signals

- Per-study top-hit rate must fall in the reported 54.31–85.51% range for the four public studies (ST001154 61.85%, ST001683 54.31%, MTBLS1684 70.58%, MTBLS2542 85.51%).
- Per-study top-five-hit rate must fall in the reported 90.58–100% range (ST001154 96.90%, ST001683 90.58%, MTBLS1684 100%, MTBLS2542 99.29%).
- Individual peak tables must report only molecular formulas with median rank ≤ 5 when computing recovery rates.
- Formula assignments should be supported by tandem mass spectrometry data where available in the deposited annotations.
- The aligned annotated peak table must be generated with top rank ≤ 30 and number of hits ≤ 30 to avoid truncating candidate formula lists before ranking statistics are computed.

## Limitations

- Recovery rates depend on the chosen IPDB; using a different target chemical space (e.g. RefMetDB instead of ExposomeDB) changes the achievable top-hit and top-five percentages.
- MSI 1–3 annotations are not uniformly complete across public studies; peaks lacking annotations cannot be scored as true positives and are excluded from the recovery rate denominator.
- IDSL.UFA only uses MS1 data; formula assignments are not corroborated by MS2 fragmentation during the benchmark, so multi-isomer or multi-formula ambiguities remain.
- Version of IDSL.UFA / IDSL.IPA is not stated in the discussion section, so exact package versions used for the published numbers are not traceable from the article alone.

## Evidence

- [results] Per-study top-hit and top-5 recovery rates are derived from formula rankings on aligned annotated peak tables: "at least one molecular formula having a median rank of ≤ 5"
- [results] Formula candidate lists must be capped at 30 before computing recovery statistics: "199,837 unique molecular formulas on the aligned table (top rank ≤ 30 and number of hits ≤ 30)"
- [results] MSI 1–3 annotations for the benchmark studies are deposited on Zenodo and serve as ground-truth true positives: "These studies have reported annotations with MSI 1–3 confidence levels (https://zenodo.org/record/5838709)"
- [results] The IPDB target space used for annotation comes from IDSL.ExposomeDB with a fixed number of ion formulas per ionization mode: "These results were generated using the IPDB of the IDSL.ExposomeDB with 209,592 and 129,122 ion formulas in positive and negative modes"
- [readme] IDSL.IPA README documents the quick-batch workflow that produces the peaklists and peak_alignment directories consumed by IDSL.UFA: [evidence span withheld — anchor status: not_found]
- [readme] IDSL.UFA README documents the UFA_workflow entry point that runs the parameter spreadsheet end-to-end: [evidence span withheld — anchor status: not_found]
