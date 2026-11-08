---
name: pfas-mass-defect-prioritization
description: Use when non-target HRMS data (mzML, ddMS2, centroided, ESI/APCI) yields
  thousands of features and you need to separate PFAS analytes from matrix components
  using mass-defect, KMD, MS2-fragment, and suspect-list criteria.
when_to_use_negative:
- The input is raw vendor data not converted to mzML, or spectra are profile rather
  than centroided — PFΔScreen requires vendor-independent centroided mzML from ddMS2
  acquisition.
- The analysis is a routine target quantification of known analytes with authentic
  standards, not non-target feature prioritization.
- Only MS1 data are available and no corresponding mzML with MS2 exists while custom
  feature lists are used — MS2-based diagnostic-fragment evaluation would be missing.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PFΔScreen
  role: 'Primary tool executing the prioritization workflow: FeatureFinding, MD/C-m/C
    reduction, KMD analysis, fragment matching, suspect screening, and HTML plot/Excel
    export via its GUI (''Run PFASPrioritization'' button)'
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs feature detection from vendor-independent mzML raw data prior to
    prioritization
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d files) into centroided mzML with
    ddMS2 for PFΔScreen input
- name: OpenMS
  role: Provides the metabolites (5% RMS) isotope model used during feature detection
    to exclude features with unusual isotopic peak shapes
- name: CompTox Dashboard
  role: Source of PFAS entries (e.g., PFASOECDNA list coverage) for extending the
    custom suspect list
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:fbe00d4977a37306b50f4d891fae4b7d7c75f69d383d086f3c1c436e96f83c1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - CompTox Dashboard
  - MSConvert
  - OpenMS
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pfas-mass-defect-prioritization

## Summary

Reduces thousands of HRMS features from non-target LC/GC-HRMS data to a manageable list of potential PFAS by combining the MD/C-m/C approach, Kendrick mass defect analysis of homologous series, MS2 diagnostic-fragment and fragment-mass-difference matching, and accurate-mass suspect screening, as implemented in the open-source Python tool PFΔScreen.

## When to use

Use when non-target HRMS data (mzML, ddMS2, centroided, ESI/APCI) yields thousands of features and you need to separate PFAS analytes from matrix components using mass-defect, KMD, MS2-fragment, and suspect-list criteria.

## When NOT to use

- The input is raw vendor data not converted to mzML, or spectra are profile rather than centroided — PFΔScreen requires vendor-independent centroided mzML from ddMS2 acquisition.
- The analysis is a routine target quantification of known analytes with authentic standards, not non-target feature prioritization.
- Only MS1 data are available and no corresponding mzML with MS2 exists while custom feature lists are used — MS2-based diagnostic-fragment evaluation would be missing.

## Inputs

- Sample mzML file (ddMS2, centroided spectra, ideally one collision energy per precursor)
- Optional blank mzML file (e.g., extraction blank) for blank correction
- Feature list from the FeatureFinding tab (pyOpenMS) or an external feature table (external_feature_list.xlsx) with the corresponding mzML file
- Custom suspect list (.csv, NIST-derived, extensible with NORMAN/CompTox entries)

## Outputs

- Prioritized results table (Excel format) containing MD/C, m/C, KMD series assignment, fragment/diagnostic-fragment hits, and suspect hits
- Interactive HTML plots: MD/C-m/C plot, m/z vs. RT plot (with and without MS2 raw data), KMD vs. m/z linked to m/z vs. RT, and m/C histogram
- MS2 spectra in RawDataVisualization with highlighted fragment mass differences and diagnostic fragments
- Optionally overlaid theoretical isotope patterns of suspect hits on experimental MS1 patterns

## How to apply

Load the FeatureFinding-tab output (sample and optionally blank feature list) together with the centroided ddMS2 mzML file into PFΔScreen, since MS2-based evaluation requires the raw data. Apply the MD/C-m/C data reduction by deriving carbon counts from the M+1 isotope and filtering with the m/C cutoff of 30 and the MD/C cutoff of < +0.003. Run KMD analysis with PFAS-representative repeating units such as (CF2)n and CF2O, flagging homologous series within ±2 mDa with at least 3 homologues. Match MS2 diagnostic fragments and fragment mass differences per polarity at ±2 mDa with an MS2 intensity threshold of 2000 counts, and screen accurate masses against the custom extensible .csv suspect list (NIST-derived, extendable with NORMAN/CompTox entries) using a 4 mDa tolerance and [M-H]-, [M+H]+, and [M]+ adducts. Export the prioritized Excel results table and interactive HTML plots (MD/C-m/C plot, m/z vs. RT with/without MS2, KMD vs. m/z linked to m/z vs. RT, m/C histogram) into a sample-named folder; the short runtime (<1 min for ~4000 spectra) permits convenient parameter adjustment and re-running.

## Related tools

- **PFΔScreen** (Primary tool executing the prioritization workflow: FeatureFinding, MD/C-m/C reduction, KMD analysis, fragment matching, suspect screening, and HTML plot/Excel export via its GUI ('Run PFASPrioritization' button)) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs feature detection from vendor-independent mzML raw data prior to prioritization) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d files) into centroided mzML with ddMS2 for PFΔScreen input)
- **OpenMS** (Provides the metabolites (5% RMS) isotope model used during feature detection to exclude features with unusual isotopic peak shapes)
- **CompTox Dashboard** (Source of PFAS entries (e.g., PFASOECDNA list coverage) for extending the custom suspect list)

## Examples

```
double-click Run_PFAScreen.bat; in the GUI set PFASPrioritization parameters and press "Run PFASPrioritization"
```

## Evaluation signals

- The exported Excel results table and all named HTML plots (MD/C-m/C, m/z vs. RT with/without MS2, KMD vs. m/z, m/C histogram) exist in the folder named after the sample.
- The results table contains the prioritization fields: MD/C, m/C, KMD series assignment, fragment/diagnostic-fragment hits, and suspect hits.
- MS2 spectra rendered by RawDataVisualization show highlighted fragment mass differences and diagnostic fragments where detected.
- Feature-reduction is consistent with the applied cutoffs: features removed by m/C = 30 and MD/C < +0.003, KMD series flagged at ±2 mDa with ≥3 homologues, suspect hits at 4 mDa.
- Runtimes stay below ~1 minute for ~4000 spectra per sample; substantially longer runs suggest misconfiguration.

## Limitations

- PFΔScreen version number is not stated, runtime claims lack hardware specifications, and no default/recommended input parameter values are documented.
- The suspect list is custom and extensible, so prioritization coverage depends on the list used (e.g., NIST-derived, NORMAN/CompTox); the number of unknown PFAS in complex samples remains unknown.
- Feature detection and cutoffs (10 ppm mass error, 2000-count MS1 threshold, FWHM 1 s–1 min, ≥2 isotopic traces, OpenMS isotope model) must be met before prioritization; peaks failing these criteria are never evaluated.
- Manual verification of each feature for blank occurrence and peak shape (until <1% of the most abundant feature) is still required after automated prioritization.

## Evidence

- [abstract] Feature prioritization using PFAS-specific techniques: "to perform efficient feature prioritization using several PFAS-specific techniques such as the highly promising MD/C-m/C approach, Kendrick mass defect analysis, diagnostic fragments (MS 2 ),"
- [results] KMD parameters: "repeating units representative for PFAS such as (CF2)n and CF2O were applied to detect HS (mass tolerance was set at ± 2 mDa, with at least 3 homologues)"
- [results] MD/C-m/C cutoffs: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] Suspect screening tolerance: "the hits by accurate mass (tolerance of 4 mDa) were reduced from 217 to 176"
- [readme] Output artifacts: "After executing the PFASPrioritization tab, the PFΔScreen results table (Excel format) and several interactive HTML plots are saved in a folder named after the sample"
- [results] Runtime claim: "PFΔScreen runtimes are usually below 1 min (e.g., for  ~ 4000 spectra per sample) for the whole workflow."
- [results] MS2 fragment matching: "at a mass tolerance set to ± 2 mDa and an MS2 intensity threshold of 2000 counts in the M1 soil extract (first iteration)"
