---
name: dbdi-plasma-ionization-in-source-fragment-identification
description: Use when a direct-injection DBDI-MS (or other plasma ionization) dataset
  has been aligned to a scan-by-feature XIC table and the goal is to flag which features
  are rivaling ion species (adducts/in-source fragments) of the same compound rather
  than distinct metabolites.
when_to_use_negative:
- Data were acquired with chromatographic separation (LC-MS) — DBDIpy is specifically
  tailored to direct-injection, time-resolved plasma ionization datasets where missing
  chromatographic separation complicates ion-species interpretation.
- The scan series is not time-resolved (single-spectrum or non-sequential scans),
  so pointwise correlation of XIC traces — the core of the detection — cannot be computed.
- The dataset is already an annotated feature table where rivaling ion species have
  been resolved; re-running identify_adducts would duplicate existing adduct annotations.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: DBDIpy
  role: Provides align_spectra(), impute_intensities() and identify_adducts() — the
    pointwise-correlation plus mass-difference search that flags in-source fragments/adducts
    in time-resolved DBDI-MS feature tables
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Imports raw spectra from .mgf/.mzML/.mzXML into matchms.Spectra objects that
    align_spectra() consumes; also hosts the matchms ecosystem interface DBDIpy builds
    on
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Data structure for the aligned two-dimensional feature table (one column per
    scan, mean m/z first column) that identify_adducts() processes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:a989dd9b9ef72079caa4a9c5db544b04da797a3133971c720a6e0e2f9852fa45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# dbdi-plasma-ionization-in-source-fragment-identification

## Summary

Putatively identifies in-source fragments (e.g. [M-H2O+H]+) and adducts (e.g. [M+nO+H]+) of the same analyte in time-resolved, direct-injection plasma ionization MS1 data, using DBDIpy's pointwise XIC correlation combined with exact mass-difference filtering within a 2 ppm window.

## When to use

Use when a direct-injection DBDI-MS (or other plasma ionization) dataset has been aligned to a scan-by-feature XIC table and the goal is to flag which features are rivaling ion species (adducts/in-source fragments) of the same compound rather than distinct metabolites.

## When NOT to use

- Data were acquired with chromatographic separation (LC-MS) — DBDIpy is specifically tailored to direct-injection, time-resolved plasma ionization datasets where missing chromatographic separation complicates ion-species interpretation.
- The scan series is not time-resolved (single-spectrum or non-sequential scans), so pointwise correlation of XIC traces — the core of the detection — cannot be computed.
- The dataset is already an annotated feature table where rivaling ion species have been resolved; re-running identify_adducts would duplicate existing adduct annotations.

## Inputs

- List of matchms.Spectra objects loaded from .mgf, .mzML or .mzXML via matchms.importing (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089)
- Aligned scan-by-feature pandas DataFrame of XICs with missing scans imputed (uniform-length XICs)
- Feature m/z vector (first 'mean' m/z column of the aligned table)
- Optional user-defined custom adduct/in-source fragment rules for the open search

## Outputs

- Dictionary keyed by adduct type, each holding a result DataFrame with base_mz, base_index, match_mz, match_index, mzdiff and corr columns
- Exported adduct-annotation table (e.g. identified_adducts.csv) listing putatively related feature pairs with their mass deltas and correlations
- Optional XIC overlay plots of identified adducts via plot_adducts_XIC()

## How to apply

Load raw spectra (.mgf/.mzML/.mzXML) via matchms.importing into a list of matchms.Spectra, then build features with DBDIpy.align_spectra(spec=..., ppm_window=2), which returns a two-dimensional pandas DataFrame with one column per scan and the mean m/z as the first column. Impute missing (np.nan) scan values with DBDIpy.impute_intensities(df=..., method='linear') so every feature yields a uniform-length extracted ion chromatogram (XIC). Run DBDIpy.identify_adducts(df=..., masses=feature_mz, method='pearson', threshold=0.95, mass_error=2): the two-step search first computes pointwise Pearson correlation of XIC traces to find features with matching temporal intensity profiles, then refines candidates by exact mass differences within the 2 ppm error window; by default it searches [M-H2O+H]+, [M+O+H]+ and [M+2O+H]+ rules, and custom adduct rules can be passed to the open-search. Collect the returned dictionary of per-adduct-type result DataFrames (base_mz, base_index, match_mz, match_index, mzdiff, corr) and export, e.g. to identified_adducts.csv; optionally refine with MS2 spectral similarity scoring (V2 behavior) and visualize with plot_adducts_XIC().

## Related tools

- **DBDIpy** (Provides align_spectra(), impute_intensities() and identify_adducts() — the pointwise-correlation plus mass-difference search that flags in-source fragments/adducts in time-resolved DBDI-MS feature tables) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Imports raw spectra from .mgf/.mzML/.mzXML into matchms.Spectra objects that align_spectra() consumes; also hosts the matchms ecosystem interface DBDIpy builds on) — https://github.com/matchms/matchms
- **pandas** (Data structure for the aligned two-dimensional feature table (one column per scan, mean m/z first column) that identify_adducts() processes)

## Examples

```
identified = dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.95, mass_error=2)
```

## Evaluation signals

- On the demo dataset (4196-feature wheat bread roasting DBDI-MS data, Zenodo DOI 10.5281/zenodo.7221089), the total number of identified potential adducts should be 710, i.e. ~17% of all features.
- Reported mzdiff values must equal exact adduct deltas: 15.99491 for +O, 31.98983 for +2O, 18.01057 for +H2O.
- All reported correlation coefficients must exceed the chosen Pearson threshold (r > 0.95).
- After impute_intensities(), every feature must yield a uniform-length XIC (no residual np.nan gaps across scans).
- Each result DataFrame row should pair a base_mz with a match_mz whose difference falls inside the 2 ppm mass-error window.

## Limitations

- Identifications are putative: missing chromatographic separation in direct-injection data makes data evaluation ambiguous without correlation-plus-mass-difference evidence.
- DBDIpy's default open search only covers [M-H2O+H]+, [M+O+H]+ and [M+2O+H]+ unless custom adduct rules are supplied.
- Runtime optimization is still under development, and Apple M1 users may hit matchms installation issues requiring manual installation.
- From V2 on, the search is being extended towards a three-step procedure adding MS2 spectral similarity scoring; results can differ across versions.

## Evidence

- [methods] Default adduct rules searched by identify_adducts: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [methods] Correlation-plus-mass-difference two-step search rationale: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [abstract] No prior tool for plasma-ionization adduct/fragment detection: "There is no tool for detection of adducts and in-source fragments from plasma ionization data yet, which makes data evaluation ambiguous."
- [readme] Pointwise correlation identifies features with matching temporal profiles: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
- [readme] Exact mass differences refine candidate ion-species pairs: "- (exact) mass differences are used to refine the nature of potential candidates."
