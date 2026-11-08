---
name: exact-mass-difference-adduct-search
description: Use when working with time-resolved DBDI-MS or other direct-injection
  MS1 feature tables (XICs per scan) where correlated features must be annotated as
  putative adducts (e.g. [M+O+H]+) or in-source fragments (e.g. [M-H2O+H]+) without
  chromatographic separation.
when_to_use_negative:
- Data is already chromatographically separated LC-MS/MS where standard adduct annotation
  tools apply — this skill targets time-resolved, direct-injection plasma ionization
  data
- Input is raw unaligned spectra with missing chromatographic-like time profiles —
  correlation-based matching requires uniform-length imputed XICs from align_spectra/impute_intensities
  first
- Exact precursor masses are unavailable or mass accuracy is far worse than 2 ppm
  — the mass-difference refinement step cannot discriminate candidate adduct rules
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DBDIpy
  role: Provides align_spectra(), impute_intensities() and identify_adducts() — the
    correlation-plus-mass-difference open search for adducts and in-source fragments
    with customizable adduct rules
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Imports .mgf/.mzML/.mzXML spectra via matchms.importing (load_from_mgf) to
    build the list of Spectra objects passed to align_spectra()
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Holds the 2D feature/XIC DataFrame and the concatenated per-adduct-type result
    tables exported to CSV
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:3bb7c083aa7a28f0278eafae0127541af6599ddec5c7096f31881231184346c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# exact-mass-difference-adduct-search

## Summary

Identifies adducts and in-source fragments in direct-injection plasma ionization MS1 datasets via DBDIpy's identify_adducts() open search, combining pointwise XIC correlation refinement with exact mass-difference rule matching (e.g. 18.010565 Da water loss, 15.99491 Da oxygen addition).

## When to use

Use when working with time-resolved DBDI-MS or other direct-injection MS1 feature tables (XICs per scan) where correlated features must be annotated as putative adducts (e.g. [M+O+H]+) or in-source fragments (e.g. [M-H2O+H]+) without chromatographic separation.

## When NOT to use

- Data is already chromatographically separated LC-MS/MS where standard adduct annotation tools apply — this skill targets time-resolved, direct-injection plasma ionization data
- Input is raw unaligned spectra with missing chromatographic-like time profiles — correlation-based matching requires uniform-length imputed XICs from align_spectra/impute_intensities first
- Exact precursor masses are unavailable or mass accuracy is far worse than 2 ppm — the mass-difference refinement step cannot discriminate candidate adduct rules

## Inputs

- Raw MS1 spectra file in .mgf, .mzML or .mzXML format (e.g. example_dataset.mgf)
- List of matchms.Spectra objects from load_from_mgf
- Feature-aligned pandas DataFrame of XICs from DBDIpy.align_spectra with ppm_window=2
- Imputed intensity DataFrame from DBDIpy.impute_intensities(method='linear') with mean m/z column as feature masses
- Optional user-defined adduct rules (deltamz values and motives, e.g. deltamz=47.984744, motive='O3')

## Outputs

- Dictionary of per-adduct-type pandas DataFrames with columns base_mz, base_index, match_mz, match_index, mzdiff, corr
- Concatenated adduct annotation table (e.g. identified_adducts.csv / adduct_annotations.csv) with ~17% (710/4196) of demo features annotated

## How to apply

Load raw spectra (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089) via matchms.load_from_mgf and align them to a 2D pandas feature DataFrame with dbdi.align_spectra(spec=spectrums, ppm_window=2), then impute missing intensities to uniform-length XICs with dbdi.impute_intensities(method='linear'). Run dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.9-0.95, mass_error=2): step one computes pointwise Pearson correlation to keep feature pairs with matching temporal intensity profiles, step two refines them by exact mass differences against default rules ([M-H2O+H]+, [M+1O+H]+, [M+2O+H]+) within the 2 ppm window. Extend the rule set by passing custom_adducts, e.g. {'deltamz': [47.984744], 'motive': ['O3']}, to search user-defined ion species. Collect the returned dictionary of per-adduct-type DataFrames (columns base_mz, base_index, match_mz, match_index, mzdiff, corr), concatenate them, and export as CSV. Rationale: in the absence of chromatography, co-eluting analytes can only be linked by shared temporal profiles plus diagnostic exact mass deltas such as 15.99491 Da (+O), 31.98983 Da (+2O), and 18.01057 Da (water loss).

## Related tools

- **DBDIpy** (Provides align_spectra(), impute_intensities() and identify_adducts() — the correlation-plus-mass-difference open search for adducts and in-source fragments with customizable adduct rules) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Imports .mgf/.mzML/.mzXML spectra via matchms.importing (load_from_mgf) to build the list of Spectra objects passed to align_spectra()) — https://github.com/matchms/matchms
- **pandas** (Holds the 2D feature/XIC DataFrame and the concatenated per-adduct-type result tables exported to CSV)

## Examples

```
identified = dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.9, mass_error=2, custom_adducts={'deltamz': [47.984744], 'motive': ['O3']})
```

## Evaluation signals

- Every returned mzdiff matches an exact rule delta within the 2 ppm mass error window (e.g. ~15.99491 Da for +O, ~31.98983 Da for +2O, ~18.01057 Da for water loss)
- All reported correlation coefficients exceed the chosen Pearson threshold (0.9 or 0.95)
- Each user-defined adduct type yields its own result entry in the returned dictionary of DataFrames
- Baseline reproduction: 710 potential adducts from 4196 demo features (~17%) using default [M-H2O+H]+, [M+O+H]+, [M+2O+H]+ rules
- Known high-confidence pairs reproduced, e.g. features 55 and 99 annotated with correlated [M+1-3O+H]+ ions at r > 0.9

## Limitations

- Annotations are putative only — in-source fragments and adducts are identified from predefined/customized rules without structural confirmation
- MS2 spectral similarity refinement is a planned V>2 feature (algorithm described as a three-step procedure 'in preparation'), so V2 results rest on correlation plus mass difference alone
- Identification depends on exact mass differences and temporal correlation; rivaling ionization pathways of the same analyte with weakly correlated XICs may be missed, and Apple M1 users may hit matchms installation issues requiring manual dependency installation

## Evidence

- [methods] we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process.: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [readme] custom adduct species passed to open search: "Custom adduct species can be defined by the user and passed to this open-search algorithm."
- [methods] features 55 and 99 putative adduct ions: "features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset."
- [methods] impute_intensities ensures uniform-length XICs: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
- [methods] default rules searched by identify_adducts: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
