---
name: vinsmoc-variable-mode-spectral-search
description: Use when query spectra (MGF, mzXML, or mzML) may come from molecule variants
  — precursor m/z differs from any database molecule by an explainable modification
  delta — and you need variant identification plus modification-site localization.
when_to_use_negative:
- The precursor mass of the query spectrum exactly matches a known database molecule
  — use VInSMoC's exact search mode instead of variable mode
- Spectra were already peak-merged and peak-filtered before upload — in that case
  the server's default preprocessing settings should be adjusted rather than left
  at defaults
- A generalized-peptide search is inappropriate because the candidates are generic
  small molecules (or vice versa) — the fragmentation mode must match the molecule
  class
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: Mass spectral search tool that scores query spectra against candidate molecules;
    run in variable mode to match spectra to molecule variants explaining the precursor
    mass difference
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web server hosting VInSMoC where projects, data uploads, task configuration,
    and finished-task results are managed
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:adc747ff76758e17bf133738ee36823ea7b3c2fcf6c9291c841c285e39538052
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# vinsmoc-variable-mode-spectral-search

## Summary

Run VInSMoC on the npanalysis.org server in variable search mode so query mass spectra match candidate molecules whose variants explain the precursor mass difference, then extract and evaluate the predicted modification sites.

## When to use

Use when query spectra (MGF, mzXML, or mzML) may come from molecule variants — precursor m/z differs from any database molecule by an explainable modification delta — and you need variant identification plus modification-site localization.

## When NOT to use

- The precursor mass of the query spectrum exactly matches a known database molecule — use VInSMoC's exact search mode instead of variable mode
- Spectra were already peak-merged and peak-filtered before upload — in that case the server's default preprocessing settings should be adjusted rather than left at defaults
- A generalized-peptide search is inappropriate because the candidates are generic small molecules (or vice versa) — the fragmentation mode must match the molecule class

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format (e.g. a file from MSV000086287)
- Candidate chemical database (e.g. NPAtlas preloaded on the server, or GNPSLIB base compounds with 107 frequent small-molecule modifications applied)
- npanalysis.org account and project with the uploaded, parsed data associated

## Outputs

- Task results page listing the best hit per input scan exceeding the minimum score threshold, each hit linked to its source database molecule
- Predicted modification sites per variable-mode hit (for site-localization analysis)
- Tabulated localization fractions, e.g. vinsmoc_site_localization.tsv

## How to apply

Create an account on run.npanalysis.org, create a project, upload the mass spectral input (MGF, mzXML, or mzML, e.g. from MSV000086287) as 'Mass Spectrum' data, and wait for parsing to finish before associating it to the project. Create a VInSMoC task, select the spectrum file and chemical database (e.g. NPAtlas or a custom GNPSLIB base set with 107 modification motifs applied), and set search mode to 'variable' with fragmentation mode (Metabolite or Peptide) matched to the molecule class, ion mode matched to the input spectrum, maximum absolute charge 1 ([M+H]+/[M-H]-) or 2 ([M+2H]++/[M-2H]--), and default precursor/product ion tolerances and minimum score. Leave advanced peak-processing settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) at defaults unless the spectra were preprocessed before upload, since peak merging combines peaks within the Peak Merge Threshold and peak filtering keeps only the Peaks Per Window most intense peaks per Peak Filtering Window. Submit, monitor pending/running states until finished, then read the task page reporting the best hit per input scan that exceeded the minimum score, each linked to its source database entry. For localization benchmarks, extract the predicted modification site per variable-mode hit, compute average atom distance to the correct site, and tabulate fractions within distance 2 and 3.

## Related tools

- **VInSMoC** (Mass spectral search tool that scores query spectra against candidate molecules; run in variable mode to match spectra to molecule variants explaining the precursor mass difference) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web server hosting VInSMoC where projects, data uploads, task configuration, and finished-task results are managed) — https://run.npanalysis.org

## Evaluation signals

- Reproduce the Extended Data Fig. 2 benchmark: 42% of 4,966 modified GNPSLIB structures (2,088) have the VInSMoC-predicted modification site within average atom distance ≤2 of the correct site
- 68% of 4,966 entries (3,377) have the predicted site within average distance ≤3 under default fragmentation settings
- Every reported hit exceeds the configured minimum score, and at most one best hit is returned per input scan
- Uploaded data reaches a successfully parsed state (not error) before task submission; malformed inputs with mismatched extensions fail parsing
- Task transitions pending → running → finished, and each hit's molecule links back to its entry in the original chemical database

## Limitations

- Task runtime depends on input size; small examples finish almost immediately but larger inputs may spend significant time in the Running state
- Data points are not immediately usable after upload — parsing must complete first, and malformed files (mismatched declared extension) show an error status
- Advanced preprocessing settings (peak merging/filtering) should be left unchanged unless the user has already preprocessed the spectra, otherwise defaults may mis-handle the input

## Evidence

- [methods] Benchmark reproducing site-localization fractions under default settings.: [evidence span withheld — anchor status: not_found]
