---
name: false-discovery-rate-curve-analysis
description: Use when validating a mass spectral database search (e.g. VInSMoC against
  COCONUT or NPAtlas) by recomputing hit lists across score and P-value cutoffs and
  plotting false discovery rate curves.
when_to_use_negative:
- The search was run in exact mode only — exact-mode identifications are not variant
  calls, so mass-shift decomposition and variant-mode FDR analysis do not apply.
- Input spectra have not been parsed or the task has not finished (pending/running
  states) — no hit list exists yet to sweep thresholds over.
- The goal is peak-level quality control (peak merging / peak filtering parameters)
  rather than identification-level statistical validation.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Produces the scored spectrum–molecule matches (with minimum score and MCMC
    P-value estimates) whose hit lists are re-swept to build the FDR curves
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts the VInSMoC tasks; results are collected from the finished task page
    for threshold sweep and FDR curve plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:9a636cc26673b8e62be4dfa38e7e798ef144e4fb5694d92986bea1fe3589df5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# false-discovery-rate-curve-analysis

## Summary

Characterize how the fraction of false identifications changes as spectrum–molecule match score and MCMC-estimated P-value thresholds are tightened, to validate that VInSMoC-style variable-mode database search results are statistically controlled.

## When to use

Use when validating a mass spectral database search (e.g. VInSMoC against COCONUT or NPAtlas) by recomputing hit lists across score and P-value cutoffs and plotting false discovery rate curves.

## When NOT to use

- The search was run in exact mode only — exact-mode identifications are not variant calls, so mass-shift decomposition and variant-mode FDR analysis do not apply.
- Input spectra have not been parsed or the task has not finished (pending/running states) — no hit list exists yet to sweep thresholds over.
- The goal is peak-level quality control (peak merging / peak filtering parameters) rather than identification-level statistical validation.

## Inputs

- VInSMoC task results (best hit per input scan exceeding the minimum score threshold, with precursor–molecule mass differences)
- Mass spectral input file (MGF, mzXML, or mzML, e.g. from MSV000086287)
- Chemical database searched (e.g. COCONUT or NPAtlas)
- Vector of score cut-off and MCMC-estimated P-value cut-off values to sweep

## Outputs

- False discovery rate curves (fdr_curves.png) showing FDR as a function of score and P-value thresholds
- Tabulated high-frequency mass-shift distribution (mass_shift_distribution.png) for variable-mode identifications
- Filtered hit lists at each threshold combination

## How to apply

Run the database search (e.g. VInSMoC variable mode against COCONUT on run.npanalysis.org) and collect, for each input scan, the best hit exceeding the minimum score reporting threshold along with the precursor-to-molecule mass difference Δ. Then sweep the score cut-off and the MCMC-estimated P-value cut-off across a range of values, recomputing the reported hit lists at each threshold. At each threshold, estimate the false discovery rate among retained identifications — VInSMoC removes false identifications by estimating the statistical significance of spectrum–molecule matches with a Markov chain Monte Carlo strategy — and plot FDR as a function of threshold (fdr_curves.png). Correct application is judged by a monotone relationship: FDR should decrease as score/P-value thresholds are tightened, qualitatively matching the paper's Extended Data Fig. 7, while the variable-mode identifications simultaneously show high-frequency mass shifts explainable by known transformations (e.g. −2.0 oxidation, ±14.0 methylation, −16.0 deoxygenation).

## Related tools

- **VInSMoC** (Produces the scored spectrum–molecule matches (with minimum score and MCMC P-value estimates) whose hit lists are re-swept to build the FDR curves) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts the VInSMoC tasks; results are collected from the finished task page for threshold sweep and FDR curve plotting)

## Evaluation signals

- FDR curves show decreasing false discovery rate as score/P-value thresholds are tightened (qualitatively matching Extended Data Fig. 7).
- The high-frequency mass shifts of variable-mode identifications correspond to known chemical mass shifts: −2.0 oxidation, ±14.0 methylation, −16.0 deoxygenation (Extended Data Fig. 6/7).
- The number of retained identifications decreases monotonically as the minimum score and P-value cut-offs are made more stringent.
- Retained hits are the best hit per input scan that exceeded the minimum score reporting threshold, each linking to its entry in the source chemical database.

## Limitations

- FDR control depends on VInSMoC's MCMC-estimated P-values; identifications below the minimum score threshold are simply not reported, so curves can only be computed within the reported hit list.
- Mass-shift assignments (e.g. +15.9949 Da oxidation, +14.0157 Da methylation) are putative explanations of Δ between precursor mass and database molecule mass, not confirmed structures.
- The benchmark scale (483 million GNPS spectra against 87 million molecules from PubChem and COCONUT) means FDR behavior at smaller database/spectrum scales may differ.

## Evidence

- [methods] task_008 workflow on FDR sweep: [evidence span withheld — anchor status: not_found]
