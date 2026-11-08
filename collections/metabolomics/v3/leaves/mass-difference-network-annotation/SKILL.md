---
name: mass-difference-network-annotation
description: Use when a direct-injection (no chromatographic separation) DBDI-MS dataset
  contains multiple ion species from one analyte and adduct calls from identify_adducts()
  need independent confirmation via correlated XIC traces plus exact mass differences.
when_to_use_negative:
- Input data already is a curated feature table — align_spectra() and matchms.importing
  are only needed for raw m/z-intensity spectra
- The dataset has chromatographic separation; this in-silico approach is tailored
  to time-resolved direct-injection plasma ionization data where coelution-based annotation
  is unavailable
- Only a single ion species per analyte exists (no rivaling adducts/in-source fragments),
  so correlation + mass-difference refinement has nothing to pair
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DBDIpy
  role: Provides align_spectra(), impute_intensities() and identify_adducts() for
    correlation-plus-mass-difference adduct detection on DBDI-MS datasets
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Loads .mgf/.mzML/.mzXML spectra (load_from_mgf) that are passed to DBDIpy's
    align_spectra() interface
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Holds the aligned feature DataFrame (one row per feature, one column per scan)
    used for correlation and mass-difference analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:67c1d69a64a31d29850c0e4c48a233ed5189000eaa60115bd2667bc7c4754d12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mass-difference-network-annotation

## Summary

Validates adduct/in-source-fragment annotations from direct-injection DBDI-MS data by clustering features that share matching temporal intensity profiles (pointwise correlation) with systematic exact mass differences, e.g. an [M+nO+H]+ oxygenation series such as the C15H17O2N cluster confirmed by features 55 and 99 with r > 0.9.

## When to use

Use when a direct-injection (no chromatographic separation) DBDI-MS dataset contains multiple ion species from one analyte and adduct calls from identify_adducts() need independent confirmation via correlated XIC traces plus exact mass differences.

## When NOT to use

- Input data already is a curated feature table — align_spectra() and matchms.importing are only needed for raw m/z-intensity spectra
- The dataset has chromatographic separation; this in-silico approach is tailored to time-resolved direct-injection plasma ionization data where coelution-based annotation is unavailable
- Only a single ion species per analyte exists (no rivaling adducts/in-source fragments), so correlation + mass-difference refinement has nothing to pair

## Inputs

- Raw mass spectra in .mgf, .mzML or .mzXML format (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089)
- Feature metadata annotations (e.g. example_metadata.feather)
- Custom adduct rules (name, deltamz, e.g. O3 with deltamz=47.984744)

## Outputs

- Per-rule adduct result DataFrames (base_index, base_mz, match_mz, mzdiff, corr)
- Intersected oxidation-cluster feature list (e.g. features 55 and 99 for the [M+1-3O+H]+ series)
- Cluster result table (oxidation_cluster_results.csv) with base_mz, match_mz, mzdiff, corr per member

## How to apply

Import raw spectra from example_dataset.mgf via matchms.load_from_mgf and build a feature DataFrame with dbdi.align_spectra(ppm_window=2); impute missing XIC values with dbdi.impute_intensities(method='linear') so all traces have uniform length. Run dbdi.identify_adducts with method='pearson', threshold=0.9 and mass_error=2 over the default rules ([M-H2O+H]+, [M+1O+H]+, [M+2O+H]+) plus custom rules such as an O3 rule with deltamz=47.984744. Intersect the base_index sets of the O, O2 and O3 result DataFrames with np.intersect1d to extract features belonging to a full [M+1-3O+H]+ oxygenation series. Join example_metadata.feather annotations (e.g. C15H17O2-5N) to the intersected features and tabulate base_mz, match_mz, mzdiff and corr per cluster member. The rationale is that pointwise correlation of XIC traces identifies co-ionized species while exact mass differences refine their chemical relationship.

## Related tools

- **DBDIpy** (Provides align_spectra(), impute_intensities() and identify_adducts() for correlation-plus-mass-difference adduct detection on DBDI-MS datasets) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Loads .mgf/.mzML/.mzXML spectra (load_from_mgf) that are passed to DBDIpy's align_spectra() interface) — https://github.com/matchms/matchms
- **pandas** (Holds the aligned feature DataFrame (one row per feature, one column per scan) used for correlation and mass-difference analysis)

## Examples

```
import DBDIpy as dbdi; specs = list(load_from_mgf('example_dataset.mgf')); feats = dbdi.align_spectra(spec=specs, ppm_window=2); res = dbdi.identify_adducts(feats, method='pearson', threshold=0.9, mass_error=2, custom_rules=[{'name':'O3','deltamz':47.984744}])
```

## Evaluation signals

- The confirmed oxygenation series must contain features 55 and 99 with pairwise pointwise correlations r > 0.9
- Each cluster member's mzdiff must fall within the mass_error=2 tolerance of the exact rule mass difference (e.g. 47.984744 for the O3 rule)
- Cluster membership must be the intersection of base_index sets across the O, O2 and O3 result DataFrames (np.intersect1d non-empty)
- After impute_intensities(method='linear'), all XIC traces must have uniform length (no residual np.nan fields)
- Tabulated results (oxidation_cluster_results.csv) must contain base_mz, match_mz, mzdiff and corr columns for every cluster member

## Limitations

- Apple M1 users may hit matchms installation issues requiring manual installation of the dependency
- Identification is putative only — without MS2 spectral similarity refinement (V2) or metadata confirmation, adduct assignments remain ambiguous since no chromatographic separation exists
- ppm_window in align_spectra() must be set appropriately or features will be mis-aligned across scans

## Evidence

- [methods] Correlation plus mass-difference identification: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [methods] Validation criterion for the cluster: "This tells us that features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset."
- [readme] Three-step procedure rationale: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
- [methods] Default adduct rules searched: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [methods] Imputation rationale: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
