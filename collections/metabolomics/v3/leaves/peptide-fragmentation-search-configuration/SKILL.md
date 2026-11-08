---
name: peptide-fragmentation-search-configuration
description: Use when the query mass spectra (MGF, mzXML, or mzML) originate from
  generalized peptides or the analysis aims to identify peptide-derived molecules,
  and a VInSMoC task must be set to Peptide fragmentation mode with the correct ion
  mode and scoring thresholds.
when_to_use_negative:
- The query molecules are generic small molecules rather than generalized peptides
  — use Metabolite fragmentation mode instead of Peptide mode
- The mass spectrum file has not yet finished parsing on the server (data still pending)
  — it is not ready for a task until parsing completes
- The goal is preprocessing spectra rather than running a database search task — the
  peak merging and peak filtering advanced settings govern preprocessing, not fragmentation
  mode selection
tools:
- name: VInSMoC
  role: Mass spectral search tool whose task form exposes the Peptide fragmentation
    mode setting used by this skill to score query spectra against generalized peptides
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web platform hosting VInSMoC where the task form with search mode, fragmentation
    mode, ion mode, tolerances, charge, and minimum score parameters is filled and
    submitted
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:1caaf2c416432177cc2d99634f8cdb1ca784344a9040e2d28979ec58f39fd638
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Peptide-fragmentation search configuration

## Summary

Configuring VInSMoC database-search tasks in Peptide fragmentation mode so spectra from generalized peptides are scored against candidate molecules with peptide-specific fragmentation rather than the generic Metabolite mode, including matching ion mode, charge adducts, and tolerance settings to the input.

## When to use

Use when the query mass spectra (MGF, mzXML, or mzML) originate from generalized peptides or the analysis aims to identify peptide-derived molecules, and a VInSMoC task must be set to Peptide fragmentation mode with the correct ion mode and scoring thresholds.

## When NOT to use

- The query molecules are generic small molecules rather than generalized peptides — use Metabolite fragmentation mode instead of Peptide mode
- The mass spectrum file has not yet finished parsing on the server (data still pending) — it is not ready for a task until parsing completes
- The goal is preprocessing spectra rather than running a database search task — the peak merging and peak filtering advanced settings govern preprocessing, not fragmentation mode selection

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format uploaded to the npanalysis.org server and associated to a project
- A chemical database (e.g. NPAtlas or GNPSLIBvNPAtlas) selected from the Input Data dropdown

## Outputs

- A finished VInSMoC task whose results page lists the best hit per input scan exceeding the minimum score threshold, each molecule linked to its source chemical database entry
- For benchmarking: runtime_benchmark.csv recording wall-clock runtime per (search mode, precursor ion tolerance) combination in Peptide fragmentation mode

## How to apply

In the VInSMoC task form on run.npanalysis.org, after selecting the input mass spectrum file and chemical database, set Fragmentation mode to Peptide rather than Metabolite, because Peptide mode scores spectra specifically against generalized peptides while Metabolite mode handles generic molecules. Set the ion mode (positive or negative) to match the input spectrum, since it determines which adducts are applied to convert m/z to masses. Leave precursor and product ion tolerances at defaults unless the instrument accuracy is known, and set the maximum absolute charge (1 yields [M+H]+/[M-H]-, 2 additionally allows [M+2H]++/[M-2H]--) according to expected adducts. Decide between exact search (candidate molecule mass must match precursor m/z) and variable search (variants may explain the precursor mass difference Δ), then set a minimum score that controls which hits are reported. For runtime comparisons, as in the GNPSLIBvNPAtlas benchmark, run one task per precursor ion tolerance for each search mode under the Peptide fragmentation setting and record wall-clock time from submission to the finished state.

## Related tools

- **VInSMoC** (Mass spectral search tool whose task form exposes the Peptide fragmentation mode setting used by this skill to score query spectra against generalized peptides) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web platform hosting VInSMoC where the task form with search mode, fragmentation mode, ion mode, tolerances, charge, and minimum score parameters is filled and submitted) — https://run.npanalysis.org

## Examples

```
Via run.npanalysis.org task form: Method=VInSMoC; Input Data = spectrum file (mzML) + NPAtlas; Task Parameters: Search mode=variable, Fragmentation mode=Peptide, Ion mode=positive, Min score=default
```

## Evaluation signals

- The task form shows Fragmentation mode set to Peptide and ion mode matching the uploaded spectrum (e.g. positive for a positive-mode mzML file)
- The task transitions pending → running → finished without an error state, confirming the input file was parsed and matched its declared extension
- The results page reports the best hit per input scan that exceeded the minimum score reporting threshold, and each molecule links to its entry in the source chemical database
- In a benchmark, one finished task exists per (search mode, precursor ion tolerance) combination, and recorded runtimes from runtime_benchmark.csv show VInSMoC in peptide-fragmentation exact and variable settings following a consistent trend

## Limitations

- Fragmentation mode is a binary choice — Metabolite mode handles generic molecules, so a wrongly selected mode scores spectra against the wrong molecule class
- Ion mode must match the input spectrum; a mismatch applies the wrong adducts ([M+H]+ vs [M-H]-) when converting m/z to masses
- Precursor and product ion tolerances and the advanced peak merging/filtering settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) should be left at defaults unless preprocessing has already been done, otherwise scoring may be skewed

## Evidence

- [methods] Run VInSMoC in exact search mode, sweeping the precursor ion tolerance over a series of increasing values and submitting one task per tolerance.: [evidence span withheld — anchor status: not_found]
- [methods] Create a new VInSMoC task selecting Peptide fragmentation mode and the ion mode matching the input spectrum (positive): [evidence span withheld — anchor status: not_found]
