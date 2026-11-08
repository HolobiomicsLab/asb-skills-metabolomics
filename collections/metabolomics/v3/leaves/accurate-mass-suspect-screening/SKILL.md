---
name: accurate-mass-suspect-screening
description: Use when prioritizing HRMS features (mzML, ddMS2) for potential PFAS
  and you want to flag those matching known suspect compounds by exact mass, e.g.
  NIST- or CompTox-derived lists.
when_to_use_negative:
- No suspect list is available or the compounds of interest are fully unknown — use
  KMD and diagnostic-fragment prioritization instead, since suspect screening only
  flags known exact masses
- Raw data are not centroided ddMS2 mzML (e.g. profile-mode or vendor-proprietary
  .d files not converted via MSConvert) — convert or preprocess first
- Goal is quantitation or confirmation with authentic reference standards rather than
  screening-level prioritization — accurate-mass matching alone provides only tentative
  assignment
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PFΔScreen
  role: Performs the accurate-mass suspect screening within its PFASPrioritization
    tab, using a custom extensible .csv suspect list and [M-H]-, [M+H]+, [M]+ adducts
  repo: https://github.com/JonZwe/PFAScreen
- name: Python
  role: Language in which PFΔScreen and its suspect-screening module are implemented
    (installable via Installation.bat / requirements.txt)
  repo: https://github.com/JonZwe/PFAScreen
- name: CompTox Dashboard
  role: Source of extensible suspect-list entries (e.g. PFASOECDNA list covering 92%
    of PFAS) to supplement the NIST-derived list
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:a48772b79b1b59f264179e75465144f892090d458b708ce304e81d98120cc07a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - CompTox Dashboard
  - PFΔScreen
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# accurate-mass-suspect-screening

## Summary

Accurate-mass suspect screening of MS1 features against a custom, extensible PFAS suspect list (.csv, NIST-derived and extendable with NORMAN/CompTox entries) as implemented in the PFΔScreen PFASPrioritization module, flagging prioritized features whose exact masses match suspect compounds within a 4 mDa tolerance.

## When to use

Use when prioritizing HRMS features (mzML, ddMS2) for potential PFAS and you want to flag those matching known suspect compounds by exact mass, e.g. NIST- or CompTox-derived lists.

## When NOT to use

- No suspect list is available or the compounds of interest are fully unknown — use KMD and diagnostic-fragment prioritization instead, since suspect screening only flags known exact masses
- Raw data are not centroided ddMS2 mzML (e.g. profile-mode or vendor-proprietary .d files not converted via MSConvert) — convert or preprocess first
- Goal is quantitation or confirmation with authentic reference standards rather than screening-level prioritization — accurate-mass matching alone provides only tentative assignment

## Inputs

- PFΔScreen FeatureFinding output (sample feature list, optionally blank-corrected)
- Corresponding centroided ddMS2 mzML file of the sample
- Custom PFAS suspect list in .csv format (NIST-derived; extensible with NORMAN/CompTox entries such as the PFASOECDNA list)

## Outputs

- Prioritized results table (Excel) containing suspect-hit assignments per feature
- Suspect hits whose theoretical isotope patterns can be overlaid on experimental MS1 patterns
- Suspect-hit annotations reflected in the interactive HTML output plots in the sample-named results folder

## How to apply

After FeatureFinding (pyOpenMS feature detection from centroided ddMS2 mzML, 10 ppm mass error, 2000-count MS1 intensity threshold) and blank correction against the extraction blank (features within <2 mDa and RT <0.1 min removed unless ≥5-fold more abundant in the sample), run the PFASPrioritization step that screens accurate masses of all remaining features against the custom .csv suspect list. Use a 4 mDa accurate-mass tolerance and the adducts [M-H]-, [M+H]+, and [M]+; the article reports suspect hits being reduced from 217 to 176 at this tolerance. Optionally overlay theoretical isotope patterns of suspect hits on the experimental MS1 patterns for verification. Suspect-hit assignments are written into the prioritized Excel results table alongside MD/C, m/C, KMD-series, and diagnostic-fragment fields.

## Related tools

- **PFΔScreen** (Performs the accurate-mass suspect screening within its PFASPrioritization tab, using a custom extensible .csv suspect list and [M-H]-, [M+H]+, [M]+ adducts) — https://github.com/JonZwe/PFAScreen
- **Python** (Language in which PFΔScreen and its suspect-screening module are implemented (installable via Installation.bat / requirements.txt)) — https://github.com/JonZwe/PFAScreen
- **CompTox Dashboard** (Source of extensible suspect-list entries (e.g. PFASOECDNA list covering 92% of PFAS) to supplement the NIST-derived list)

## Examples

```
double-click Run_PFAScreen.bat; in the PFASPrioritization tab browse SampleFeatures.xlsx and the ddMS2 mzML, set 4 mDa suspect tolerance, then press "Run PFASPrioritization"
```

## Evaluation signals

- Suspect hits are recorded in the exported Excel results table using a 4 mDa accurate-mass tolerance and the [M-H]-, [M+H]+, [M]+ adducts
- Theoretical isotope patterns of suspect hits overlay plausibly on the experimental MS1 isotope patterns when visualized
- Hit counts behave sensibly under tolerance reduction (article: 217 hits reduced to 176 at 4 mDa)
- All HTML output plots and the Excel table exist in the folder named after the sample after the PFASPrioritization run

## Limitations

- Accurate-mass matching is screening-level only; tentative assignments require follow-up (isotope-pattern overlay, MS2 diagnostic fragments, or reference standards)
- The default suspect list is NIST-derived and must be manually extended (e.g. with NORMAN/CompTox entries) to broaden coverage
- No default or recommended parameter values are provided; tolerances and adducts must be adjusted depending on sample, instrument, and end-user needs

## Evidence

- [results] Suspect screening accurate mass tolerance of 4 mDa: "the hits by accurate mass (tolerance of 4 mDa) were reduced from 217 to 176"
- [abstract] Suspect screening against custom extensible list: [evidence span withheld — anchor status: not_found]
- [readme] Isotope pattern overlay of suspect hits: "Also, the theoretical isotope patterns of suspect hits can be displayed over the experimental isotope patterns (MS1)."
- [readme] Custom feature lists and mzML requirement: "custom feature lists (see external_feature_list.xlsx) together with the respective mzML files can instead be included in PFΔScreen"
- [results] CompTox as suspect-list source: "which would include 92% of the PFAS in the PFASOECDNA list (CompTox Dashboard [36, 57])"
