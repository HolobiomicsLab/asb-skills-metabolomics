---
name: spectrum-molecule-match-significance-estimation
description: Use when a variable- or exact-mode VInSMoC search (e.g. GNPS spectra
  vs COCONUT/PubChem/NPAtlas) reports hits per scan and you must quantify which spectrum–molecule
  matches are statistically significant and estimate false discovery rates.
when_to_use_negative:
- You have already locally pre-processed the input spectra (Peak Merge Threshold,
  Minimum Peak Count, Peaks Per Window, Peak Filtering Window should be left unchanged
  only when spectra are NOT pre-processed) — significance estimation inherits whatever
  preprocessing artifacts exist
- The spectrum file is malformed or still in the server-side parsing/pending state
  on run.npanalysis.org — the task cannot be run or trusted until parsing succeeds
- You need an exact-match identification only with no significance assessment — the
  MCMC/FDR layer is specifically for separating true from false matches, not for producing
  hit lists alone
edam_operation: http://edamontology.org/operation_3659
tools:
- name: VInSMoC
  role: Mass spectral search tool whose matches are scored; it estimates statistical
    significance of spectrum–molecule matches via MCMC and reports hits above the
    minimum score
  repo: mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosted platform where VInSMoC tasks are configured, run, and their finished
    task pages yield the best hits per scan used for FDR analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:f3ef34e83c6bba0e6e2be280966b882f1475afd90a94beaa25979b82e7ed8b61
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectrum-molecule-match-significance-estimation

## Summary

Estimate the statistical significance of spectrum–molecule matches from VInSMoC database searches using an MCMC-based P-value strategy, and characterize target–decoy false discovery rates by sweeping score and P-value cut-offs. This is how VInSMoC separates real identifications (including molecule variants) from false matches.

## When to use

Use when a variable- or exact-mode VInSMoC search (e.g. GNPS spectra vs COCONUT/PubChem/NPAtlas) reports best hits per scan and you must quantify which spectrum–molecule matches are statistically significant and estimate false discovery rates.

## When NOT to use

- You have already locally pre-processed the input spectra (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window should be left unchanged only when spectra are NOT pre-processed) — significance estimation inherits whatever preprocessing artifacts exist
- The spectrum file is malformed or still in the server-side parsing/pending state on run.npanalysis.org — the task cannot be run or trusted until parsing succeeds
- You need an exact-match identification only with no significance assessment — the MCMC/FDR layer is specifically for separating true from false matches, not for producing hit lists alone

## Inputs

- Finished VInSMoC task results from run.npanalysis.org (best hit per input scan above the minimum score threshold)
- Per-hit mass differences between hits and their database parent molecules
- Sweep of score cut-off and MCMC-estimated P-value cut-off values
- Input mass spectra (MGF, mzXML, or mzML; e.g. a file from MSV000086287) searched against a chemical database (COCONUT, PubChem, or NPAtlas)

## Outputs

- FDR curves (fdr_curves.png) showing false discovery rate as score/P-value cut-offs are tightened
- Mass-shift distribution (mass_shift_distribution.png) of high-frequency precursor-to-molecule mass shifts
- Per-scan list of statistically significant spectrum–molecule matches with associated P-values

## How to apply

After a VInSMoC task finishes on run.npanalysis.org, collect the best hit reported for each input scan that exceeded the minimum score, together with each hit's mass difference to its database parent molecule. Then recompute the hit lists while sweeping the score cut-off and the MCMC-estimated P-value cut-off, plotting the resulting false discovery rate (FDR) curves as fdr_curves.png. Rationale: VInSMoC removes some false identifications by estimating the statistical significance of matches between spectra and molecular structures via a Markov chain Monte Carlo strategy, so tightening either the score or the P-value threshold should monotonically reduce FDR. As a domain-level validation, aggregate unique variant identifications (e.g. ~85,151 from COCONUT variable mode) and check that high-frequency mass shifts correspond to known chemical transformations such as oxidation, methylation (±14.0), and deoxygenation (−16.0), as in Extended Data Figs. 6–7. The minimum score parameter controls which hits are reported in the first place, so it is the primary knob for sensitivity vs precision.

## Related tools

- **VInSMoC** (Mass spectral search tool whose matches are scored; it estimates statistical significance of spectrum–molecule matches via MCMC and reports hits above the minimum score) — mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosted platform where VInSMoC tasks are configured, run, and their finished task pages yield the best hits per scan used for FDR analysis)

## Evaluation signals

- FDR curves show decreasing false discovery rate as score/P-value thresholds are tightened, qualitatively matching Extended Data Fig. 7
- Reproduced mass-shift distribution is dominated by the same high-frequency transformations (oxidation, methylation, deoxygenation) reported in Extended Data Fig. 6, e.g. −2.0 oxidation, ±14.0 methylation, −16.0 deoxygenation
- Hit counts of unique variant identifications are in the reported range (~85,151 from COCONUT variable mode) when using comparable cut-offs
- Every reported hit per scan exceeds the configured minimum score reporting threshold
- Database parent molecules for hits link to their original chemical database entries (COCONUT/PubChem/NPAtlas) on the task page

## Limitations

- Advanced preprocessing settings should be left unchanged unless the input spectra have already been pre-processed locally; changing them otherwise distorts the scored spectra
- Uploaded data are not immediately usable — each input is parsed server-side and may sit in a pending state (or error if the file format does not match the declared extension)
- Depending on input size, tasks can spend substantial time in the Running state before results (and hence significance estimates) are available

## Evidence

- [other] FDR curve sweep procedure: [evidence span withheld — anchor status: not_found]
- [other] Expected FDR behavior for validation: [evidence span withheld — anchor status: not_found]
