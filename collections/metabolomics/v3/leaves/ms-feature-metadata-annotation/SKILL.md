---
name: ms-feature-metadata-annotation
description: Use when a direct-injection MS dataset lacks chromatographic separation
  and features must be grouped as adducts/in-source fragments of one analyte via time-profile
  correlation plus exact mass differences.
when_to_use_negative:
- Input is already a curated feature table from LC-MS with chromatographic separation
  — retention-time-based coelution methods are more appropriate than time-profile
  correlation of direct-injection XICs.
- The goal is definitive molecular identification rather than putative adduct/in-source
  fragment annotation — correlation plus mass difference only yields putative assignments.
- MS1 data cannot be loaded via matchms-supported formats (.mgf/.mzML/.mzXML) and
  cannot first be exported with DBDIpy's export_to_spectra.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DBDIpy
  role: 'Core library: align_spectra builds the feature table, impute_intensities
    produces uniform XICs, identify_adducts performs correlation plus mass-difference
    adduct/in-source fragment detection, plot_adducts_XIC visualizes results'
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Imports MS data from .mgf/.mzML/.mzXML via matchms.importing (load_from_mgf)
    to feed DBDIpy's align_spectra
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Holds the two-dimensional feature DataFrame (features as rows, scans as columns)
    on which correlation and mass-difference analysis is computed
- name: ms2deepscore
  role: Optional MS2 spectral similarity scoring to refine candidate adduct/in-source
    fragment assignments (V2 refinement)
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:3c69fbada7fb3351ce9df5378cb89c94632a4c8a440c5e8e14c788815f7d2ce0
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# ms-feature-metadata-annotation

## Summary

Putative annotation of MS1 features as adducts or in-source fragments (e.g. [M+H]+, [M-H2O+H]+, [M+nO+H]+) arising from the same analyte, using pointwise XIC correlation combined with exact mass-difference analysis in direct-injection, chromatography-free plasma ionization (DBDI) datasets.

## When to use

Use when a direct-injection MS dataset lacks chromatographic separation and features must be grouped as adducts/in-source fragments of one analyte via time-profile correlation plus exact mass differences.

## When NOT to use

- Input is already a curated feature table from LC-MS with chromatographic separation — retention-time-based coelution methods are more appropriate than time-profile correlation of direct-injection XICs.
- The goal is definitive molecular identification rather than putative adduct/in-source fragment annotation — correlation plus mass difference only yields putative assignments.
- MS1 data cannot be loaded via matchms-supported formats (.mgf/.mzML/.mzXML) and cannot first be exported with DBDIpy's export_to_spectra.

## Inputs

- MGF, mzML or mzXML file with MS1 spectra (e.g. example_dataset.mgf from the Zenodo demo deposit DOI 10.5281/zenodo.7221089)
- Optional feature metadata table (e.g. example_metadata.feather)
- Adduct rule specification: default adducts plus custom rules with deltamz values (e.g. 47.984744 for O3)

## Outputs

- Aligned feature pandas.DataFrame with a mean m/z column and per-scan XIC intensities
- Adduct result DataFrames with base_index, base_mz, match_mz, mzdiff and corr columns per detected adduct/in-source fragment pair
- Visualizations of identified adducts (plot_adducts_XIC) and optionally tabulated cluster results (e.g. oxidation_cluster_results.csv)

## How to apply

Load raw spectra (e.g. example_dataset.mgf) via matchms.importing load_from_mgf and align them into a feature pandas.DataFrame with dbdi.align_spectra (ppm_window=2), where rows are features and the first column holds the mean m/z; missing scan values are np.nan. Impute to uniform-length extracted ion chromatograms (XIC) with dbdi.impute_intensities(method='linear'). Then run dbdi.identify_adducts, which computes pointwise correlation of XIC traces to find features with matching temporal intensity profiles and refines candidates by exact mass differences; default searches cover [M-H2O+H]+, [M+1O+H]+ and [M+2O+H]+, custom rules (e.g. an O3 rule with deltamz=47.984744) can be added, and in this validation correlation threshold 0.9 with mass_error=2 ppm was used. Custom adduct species are passed to the open-search algorithm, and results are visualized with plot_adducts_XIC(); MS2 spectral similarity scoring (V2 refinement) can further refine candidates. The rationale is that in direct injection the missing chromatographic separation makes ion-species grouping ambiguous, so time-resolved intensity co-variation is the primary discriminating signal.

## Related tools

- **DBDIpy** (Core library: align_spectra builds the feature table, impute_intensities produces uniform XICs, identify_adducts performs correlation plus mass-difference adduct/in-source fragment detection, plot_adducts_XIC visualizes results) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Imports MS data from .mgf/.mzML/.mzXML via matchms.importing (load_from_mgf) to feed DBDIpy's align_spectra) — https://github.com/matchms/matchms
- **pandas** (Holds the two-dimensional feature DataFrame (features as rows, scans as columns) on which correlation and mass-difference analysis is computed)
- **ms2deepscore** (Optional MS2 spectral similarity scoring to refine candidate adduct/in-source fragment assignments (V2 refinement)) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.importing import load_from_mgf; import DBDIpy as dbdi; specs = list(load_from_mgf('example_dataset.mgf')); feats = dbdi.align_spectra(spec=specs, ppm_window=2); feats = dbdi.impute_intensities(feats, method='linear'); res = dbdi.identify_adducts(feats, method='pearson', threshold=0.9, mass_error=2)
```

## Evaluation signals

- Result DataFrames for related adduct rules (e.g. O, O2, O3) share intersecting base_index sets, confirming a full oxygenation series (e.g. [M+1-3O+H]+) when intersected with np.intersect1d.
- Confirmed adduct pairs show pointwise XIC correlations above the threshold (r > 0.9 in the demo: features 55 and 99 putatively have [M+1-3O+H]+ adduct ions with correlations r > 0.9).
- Observed mass differences between paired features match the rule deltamz within the specified mass_error (2 ppm in the reported workflow).
- After impute_intensities, all XIC traces have uniform length, so correlation can be computed pointwise across every scan.
- Annotations joined from metadata (e.g. C15H17O2-5N) are consistent with the annotated adduct formulae in the result table.

## Limitations

- Assignments are putative only — correlation of temporal intensity profiles plus exact mass differences cannot definitively confirm structural identity without MS2 or independent validation (the C15H17O2N series required a separate network-based annotation workflow for confirmation).
- Apple M1 chip users might encounter issues with automatic installation of matchms and need manual dependency installation.
- A signal not detected in a scan is stored as np.nan, so imputation choice (method='linear') directly influences the correlation step.
- Advanced plotting options and runtime optimization are still under development in DBDIpy 2.0.

## Evidence

- [methods] Pointwise correlation plus mass-difference identification: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [methods] Default adduct search scope: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [methods] Demo validation of correlated features: "This tells us that features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset."
- [readme] Alignment to feature table: "Raw mass spectra containing *m/z*-intensity-pairs first will need to be aligned to a DataFrame of features."
- [readme] Rationale: chromatography-free ambiguity: "Mass spectrometric data from direct injection analysis is hard to interpret as missing chromatographic separation complicates identification of fragments and adducts generated during the ionization"
