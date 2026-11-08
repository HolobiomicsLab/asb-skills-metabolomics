---
name: variable-mode-molecular-variant-search-configuration
description: Use when tandem mass spectra (MGF, mzXML, or mzML) must be searched against
  a chemical database like COCONUT or NPAtlas in a mode that can explain precursor
  mass differences via known variant mass shifts, not just exact matches.
when_to_use_negative:
- When the goal is exact search of known molecules only, since exact mode already
  matches spectra to molecules whose masses match the precursor m/z without variant
  interpretation
- When the input spectra have already been locally pre-processed and peak-thresholded
  — the advanced settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window,
  Peak Filtering Window) should be adjusted rather than left at defaults
- When ion/adduct assignment requires charges above 2, since the form supports a maximum
  absolute charge of only 1 or 2
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Mass spectral search tool configured in variable mode to score query spectra
    against database molecules and interpret precursor mass differences as molecular
    variants
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: 'Web platform hosting VInSMoC: account creation, project setup, data upload/parsing,
    task submission and result viewing'
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:83c0002d6aad9a1cdde5f2eb3b4084bb8efaac396d4e4aad21f78c8f4df86f1a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# variable-mode-molecular-variant-search-configuration

## Summary

Configure and run a VInSMoC variable-mode database search on the run.npanalysis.org server so that query mass spectra are matched to database molecules whose structural variants explain the precursor-to-molecule mass difference, enabling identification of unreported molecule variants.

## When to use

Use when tandem mass spectra (MGF, mzXML, or mzML) must be searched against a chemical database like COCONUT or NPAtlas in a mode that can explain precursor mass differences via known variant mass shifts, not just exact matches.

## When NOT to use

- When the goal is exact search of known molecules only, since exact mode already matches spectra to molecules whose masses match the precursor m/z without variant interpretation
- When the input spectra have already been locally pre-processed and peak-thresholded — the advanced settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) should be adjusted rather than left at defaults
- When ion/adduct assignment requires charges above 2, since the form supports a maximum absolute charge of only 1 or 2

## Inputs

- Mass spectral input file in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- Chemical database available on the npanalysis server (COCONUT, NPAtlas, PubChem subsets)

## Outputs

- Per-scan best-hit list (best hit per input scan exceeding the minimum score, each linked to its chemical database entry)
- Mass difference (Δ) between each hit and its database parent molecule
- Variant identifications aggregatable into mass-shift frequency distributions (e.g. ~85,151 unique COCONUT variable-mode identifications)

## How to apply

Upload the mass spectral input (MGF, mzXML, or mzML, e.g. a file from MSV000086287) to run.npanalysis.org, wait for server-side parsing to finish, and associate both the spectra and the chemical database (COCONUT or NPAtlas) with a project. Create a VInSMoC task and set search mode to 'variable' — unlike exact mode, variable mode matches spectra to candidate molecules whose variants explain the mass difference Δ between the spectrum's precursor and the molecule. Set the ion mode to match the input (e.g. positive) and choose the maximum absolute charge: 1 restricts adducts to [M+H]+/[M-H]-, 2 additionally allows [M+2H]++/[M-2H]--; keep precursor and product ion tolerances at defaults if unsure, and set a minimum score as the reporting threshold. Leave the advanced preprocessing settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) unchanged unless the spectra were pre-processed locally, then submit and monitor the task through pending/running states until finished. From the finished task page, collect the best hit per scan exceeding the minimum score plus each hit's mass difference from its database parent molecule for downstream mass-shift frequency analysis.

## Related tools

- **VInSMoC** (Mass spectral search tool configured in variable mode to score query spectra against database molecules and interpret precursor mass differences as molecular variants) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web platform hosting VInSMoC: account creation, project setup, data upload/parsing, task submission and result viewing) — https://run.npanalysis.org

## Evaluation signals

- Task page shows a best hit for each input scan that exceeded the minimum score reporting threshold, with each molecule linked to its original chemical database entry
- Input data transitions from pending to successfully parsed status on the server before the task is launched (error status indicates a malformed file or extension mismatch)
- The mass difference (Δ) per hit can be computed and aggregated; high-frequency shifts should correspond to known transformations such as oxidation (+15.9949 Da), methylation (+14.0157 Da), and deoxygenation (-15.9949 Da)
- Sweeping the score cut-off and the MCMC-estimated P-value cut-off should yield FDR curves with decreasing false discovery rate as thresholds are tightened
- Adduct assignment is consistent with the ion mode: charge 1 gives [M+H]+ or [M-H]- and charge 2 additionally gives [M+2H]++ or [M-2H]--

## Limitations

- Data inputs are parsed server-side after upload and are not immediately ready for use; malformed files show an error status requiring re-upload with a corrected format/extension
- Large inputs may spend substantial time in the 'Running' state; small demo runs finish almost immediately
- Precursor and product ion tolerances determine match thresholds, and the authors note these 'can be safely left to the default' when unsure — mis-tuned tolerances affect hit quality
- Variable mode identifies molecule variants by mass shift, not full structure elucidation; variant identifications are 'previously unreported' candidates rather than confirmed structures

## Evidence
