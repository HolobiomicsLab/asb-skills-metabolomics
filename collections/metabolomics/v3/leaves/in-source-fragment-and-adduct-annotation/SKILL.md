---
name: in-source-fragment-and-adduct-annotation
description: Use when working with untargeted, time-resolved direct-injection (e.g.
  DBDI-FT-ICR-MS) MS data lacking chromatographic separation, and multiple ion species
  from one analyte (adducts/in-source fragments) must be putatively identified.
when_to_use_negative:
- Data is LC-MS with chromatographic separation, where coelution rather than temporal-profile
  correlation drives ion-species assignment
- Input is already a curated feature table without time-resolved XIC traces — the
  correlation step requires uniform-length temporal intensity profiles per feature
- Only a single ion species per analyte is expected and no adduct/fragment deconvolution
  is needed; use matchms preprocessing alone instead
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: DBDIpy
  role: Provides align_spectra(), impute_intensities() and identify_adducts() — the
    correlation + mass-difference open-search that annotates in-source adducts and
    fragments
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Interfaces for importing .mgf/.mzML/.mzXML spectra (matchms.importing) used
    as input to identify_adducts()
  repo: https://github.com/matchms/matchms
- name: ms2deepscore
  role: Optional MS2 spectral similarity scoring for automated refinement of adduct/in-source
    fragment candidates (V2 refinement step)
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:529909fffeecb64e3ea80ee1a108c8c4aac76249b93a6aedfb7579825e336c18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - ms2deepscore
  - pandas
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# in-source-fragment-and-adduct-annotation

## Summary

Putative annotation of in-source fragments (e.g. [M-H2O+H]+) and in-source generated adducts (e.g. [M+nO+H]+) in direct-injection plasma ionization MS datasets by combining pointwise correlation of XIC traces with exact mass-difference matching against a customizable rule set.

## When to use

Use when working with untargeted, time-resolved direct-injection (e.g. DBDI-FT-ICR-MS) MS data lacking chromatographic separation, and multiple ion species from one analyte (adducts/in-source fragments) must be putatively identified.

## When NOT to use

- Data is LC-MS with chromatographic separation, where coelution rather than temporal-profile correlation drives ion-species assignment
- Input is already a curated feature table without time-resolved XIC traces — the correlation step requires uniform-length temporal intensity profiles per feature
- Only a single ion species per analyte is expected and no adduct/fragment deconvolution is needed; use matchms preprocessing alone instead

## Inputs

- List of matchms.Spectra objects from example_dataset.mgf (or .mzML/.mzXML)
- Feature DataFrame of aligned, uniform-length XICs (specs_imputed)
- Feature mean m/z vector (feature_mz)
- Adduct rule set (default [M-H2O+H]+, [M+1O+H]+, [M+2O+H]+ plus custom rules with deltamz and motive)

## Outputs

- Dictionary with one DataFrame per adduct type (columns: base_mz, base_index, match_mz, match_index, mzdiff, corr)
- Adduct/in-source fragment annotations per feature (e.g. features 55 and 99 as [M+1-3O+H]+ series)
- Optional plots of XICs of identified adducts via plot_adducts_XIC()

## How to apply

Load raw MS1 spectra (.mgf/.mzML/.mzXML) via matchms.importing and align them to a feature DataFrame with align_spectra(ppm_window=2); impute missing scan values with impute_intensities(method='linear') so every feature yields a uniform-length XIC. Run identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.9, mass_error=2), which first computes pointwise Pearson correlation between all XIC pairs to find features with matching temporal profiles, then refines correlated pairs by exact mass-difference matching within the 2 ppm error. By default the rule set covers [M-H2O+H]+ (18.010565 Da), [M+1O+H]+ (15.99491 Da) and [M+2O+H]+ (31.98983 Da); user-defined rules (e.g. deltamz=47.984744, motive 'O3') extend the open search. The result is a dictionary holding one DataFrame per adduct type (columns base_mz, base_index, match_mz, match_index, mzdiff, corr), which can be visualized with plot_adducts_XIC() or joined to metadata. From V2 onward, refinement by MS2 spectral similarity scoring (and propose_adducts() for an open search) can further support candidate assignment.

## Related tools

- **DBDIpy** (Provides align_spectra(), impute_intensities() and identify_adducts() — the correlation + mass-difference open-search that annotates in-source adducts and fragments) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Interfaces for importing .mgf/.mzML/.mzXML spectra (matchms.importing) used as input to identify_adducts()) — https://github.com/matchms/matchms
- **ms2deepscore** (Optional MS2 spectral similarity scoring for automated refinement of adduct/in-source fragment candidates (V2 refinement step)) — https://github.com/matchms/ms2deepscore

## Examples

```
results = dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method="pearson", threshold=0.9, mass_error=2)
```

## Evaluation signals

- All reported Pearson correlations between paired XICs exceed the 0.9 threshold passed to identify_adducts()
- Every mzdiff falls within the 2 ppm mass error of its rule mass (e.g. O ~15.99491, O2 ~31.98983, O3 ~47.98474, H2O ~18.01057 Da)
- Paired features share matching temporal intensity profiles across all scans of the experiment
- Known analyte validation: oxidation series of C15H17O2N recovers the four-member series (features 55, 66, 83, 99) annotated [M+1-3O+H]+
- Output dictionary schema: one DataFrame per adduct type with columns base_mz, base_index, match_mz, match_index, mzdiff, corr

## Limitations

- Assignments are putative only — the in-silico approach proposes ion species without chromatographic separation, making full data evaluation ambiguous without additional evidence
- Default rule coverage is limited to [M-H2O+H]+, [M+1O+H]+ and [M+2O+H]+; custom adducts must be defined explicitly by the user
- Apple M1 users may hit automatic matchms installation issues requiring manual installation
- MS2 spectral similarity refinement is described as in preparation for V > 2; advanced plotting and runtime optimization are still under development

## Evidence

- [methods] we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [methods] By default, identify_adducts() searches for [M-H2O+H]+, [M+1O+H]+ and [M+2O+H]+.: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [methods] align_spectra() is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via matchms.importing: "``align_spectra()`` is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via ``matchms.importing``."
- [readme] calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment.: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
- [readme] (exact) mass differences are used to refine the nature of potential candidates.: "(exact) mass differences are used to refine the nature of potential candidates."
