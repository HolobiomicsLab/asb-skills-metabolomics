---
name: xic-pointwise-correlation-analysis
description: Use when the input is a time-resolved direct injection MS dataset (DBDI/plasma
  ionization, .mgf/.mzML/.mzXML) and putative adducts or in-source fragments of the
  same analyte must be identified without chromatographic separation.
when_to_use_negative:
- The data is already LC-MS with chromatographic separation — the method is tailored
  to direct-injection, time-resolved plasma ionization data (e.g. breath analysis,
  process control, food research)
- The input is raw unaligned spectra; align_spectra and impute_intensities must be
  run first, since identify_adducts expects a uniform-length XIC DataFrame
- The goal is MS2 structural identification rather than in-source species grouping
  — use ms2deepscore/spectral similarity scoring refinement instead of correlation
  alone
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3295
tools:
- name: DBDIpy
  role: Provides align_spectra(), impute_intensities() and identify_adducts() implementing
    the pointwise-correlation-plus-mass-difference open search
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Loads .mgf/.mzML/.mzXML spectra via matchms.importing (load_from_mgf) feeding
    the alignment step
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Hosts the 2D feature/XIC DataFrame (one column per scan, one row per feature)
    consumed by identify_adducts()
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:1ee5b059f4941d5a9965ad9e99f0bcbc68417f2d6b1743bbaef33b57fd8ad399
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# xic-pointwise-correlation-analysis

## Summary

Two-step open-search method from DBDIpy that computes pointwise correlation of extracted ion chromatograms (XICs) across time-resolved direct-injection DBDI-MS scans to find co-eluting features, then refines candidates by exact mass-difference matching against adduct/in-source fragment rules (e.g. [M-H2O+H]+, [M+nO+H]+).

## When to use

Use when the input is a time-resolved direct injection MS dataset (DBDI/plasma ionization, .mgf/.mzML/.mzXML) and putative adducts or in-source fragments of the same analyte must be identified without chromatographic separation.

## When NOT to use

- The data is already LC-MS with chromatographic separation — the method is tailored to direct-injection, time-resolved plasma ionization data (e.g. breath analysis, process control, food research)
- The input is raw unaligned spectra; align_spectra and impute_intensities must be run first, since identify_adducts expects a uniform-length XIC DataFrame
- The goal is MS2 structural identification rather than in-source species grouping — use ms2deepscore/spectral similarity scoring refinement instead of correlation alone

## Inputs

- List of matchms.Spectra objects loaded from an .mgf/.mzML/.mzXML file (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089)
- Feature DataFrame of aligned XICs from dbdi.align_spectra(ppm_window=2)
- Imputed XIC DataFrame from dbdi.impute_intensities(method='linear') plus the mean m/z feature masses
- User-defined adduct rules as a DataFrame with deltamz and motive columns (optional)

## Outputs

- Dictionary holding one DataFrame per adduct type with columns base_mz, base_index, match_mz, match_index, mzdiff, corr
- Putative adduct/in-source fragment assignments (e.g. features 55 and 99 as [M+1-3O+H]+ with r > 0.9)
- Tabular export (e.g. adduct_search_results.csv) of correlated feature pairs

## How to apply

Import spectra via matchms.load_from_mgf and align them to a feature DataFrame with dbdi.align_spectra(ppm_window=2), where each row is a feature with a mean m/z column. Re-add the mean m/z column as feature masses and fill missing scan values with dbdi.impute_intensities(method='linear') so all XICs have uniform length. Run dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.9, mass_error=2): step one computes pointwise Pearson correlation across all XIC pairs and keeps features whose temporal intensity profiles match (threshold=0.9); step two refines correlated pairs by exact mass-difference matching within 2 ppm against pre-defined rules — by default [M-H2O+H]+ (18.01057 Da), [M+O+H]+ (~15.99491 Da) and [M+2O+H]+, extendable with custom rules such as an O3 adduct (deltamz=47.984744, motive 'O3'). The result is a dictionary of per-adduct-type DataFrames with columns base_mz, base_index, match_mz, match_index, mzdiff, corr. The rationale is that ions of one analyte generated in the plasma source track each other's intensity over the experiment, so temporal correlation plus exact mass difference puts them into one ion-species family.

## Related tools

- **DBDIpy** (Provides align_spectra(), impute_intensities() and identify_adducts() implementing the pointwise-correlation-plus-mass-difference open search) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Loads .mgf/.mzML/.mzXML spectra via matchms.importing (load_from_mgf) feeding the alignment step) — https://github.com/matchms/matchms
- **pandas** (Hosts the 2D feature/XIC DataFrame (one column per scan, one row per feature) consumed by identify_adducts())

## Examples

```
specs_imputed = dbdi.impute_intensities(dbdi.align_spectra(spec=list(load_from_mgf(demo_mgf)), ppm_window=2), method='linear'); results = dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.9, mass_error=2)
```

## Evaluation signals

- All reported pairwise correlations of matched features exceed the correlation threshold (e.g. r > 0.9 for features 55 and 99)
- Every mzdiff falls within the 2 ppm mass error of its rule mass (e.g. O-series ~15.99491 Da, H2O loss ~18.01057 Da, O3 47.984744 Da)
- The returned object is a dictionary with one DataFrame per adduct rule, each containing the base_mz, base_index, match_mz, match_index, mzdiff, corr schema
- XICs are uniform length after imputation and no np.nan remain in the imputed DataFrame
- For a full oxygenation series, base_index sets of the O, O2 and O3 result DataFrames intersect (np.intersect1d) to yield the same analyte family (e.g. [C15H17O2N+H]+ cluster)

## Limitations

- Assignments are putative only; the correlation+mass-difference search does not prove structural identity and is best cross-validated (e.g. by network-based annotation or MS2 spectral similarity scoring)
- Default rules cover only [M-H2O+H]+, [M+O+H]+ and [M+2O+H]+; other ion species require user-defined custom adduct rules
- MS2-based refinement is part of V2 development; identify_adducts() relies on MS1 XIC correlation, and Apple M1 users may need to install matchms manually due to installation issues

## Evidence

- [methods] Pointwise correlation plus mass difference analysis: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [intro] Temporal intensity profile matching rationale: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
- [methods] Default adduct rule set: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [readme] XIC imputation requirement: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
- [methods] Correlation threshold validation anchor: "This tells us that features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset."
